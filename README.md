<p align="center">

<img src="Images/logo.jpeg" alt="Debug" title="Debug"/>

</p>

[英文](README_EN.md)

[中文](README_CN.md)

## はじめに

> デバッグには悪いイメージがあります。<br/>そんなことを考えてはいけません。<br/>ソフトウェアには常にバグが存在しますが、それはどんなソフトウェアでも同じです。<br/> <br/>そのようなことはありません。<br/>代わりに、あなたはデバッグを**プログラムをよりよく理解するためのプロセス**として捉えるべきです。<br>その代わりに、デバッグはプログラムを理解するためのプロセスであると考えるべきです。

Cobol言語の創始者であるグレース・ホッパーがリレーコンピュータに世界初のBugを発見して以来、ソフトウェア開発におけるBugの発生は止まるところを知らない。この本の序文にあるように、《Advanced Apple Debugging & Reverse Engineering》はこう語っている。開発者は、ソフトウェアがどのように動作するかをよく理解していれば、Bugが発生しないとは思いたくない。そのため、ソフトウェア開発のライフサイクルにおいて、デバッグは避けては通れない段階なのです。

## デバッグの概要

経験の浅いプログラマーに、デバッグの定義を尋ねると
 デバッグの定義を尋ねると、「デバッグとは、ソフトウェアの問題の解決策を見つけるために行うものです」と答えるかもしれません。その通りですが、それは本当のデバッグのほんの一部に過ぎません。

本当の意味でのデバッグとは、次のようなものです。
1. なぜ予期せぬ動作をしているのかを知る。
2. それを解決する
3. 新たな問題が発生していないことを確認する
4. 読みやすさ、アーキテクチャ、テストカバレッジ、パフォーマンスなど、コードの品質を向上させる。
5. 同じ問題が他の場所で発生しないようにすること

上記のステップの中で、最も重要なステップは、最初のステップである「問題の発見」です。どうやら、それは他のステップの前提条件のようです。

調査によると、経験豊富なプログラマーが同じ不具合を見つけるためにデバッグに費やす時間は、経験の浅いプログラマーの約20分の1だそうです。つまり、デバッグの経験がプログラミングの効率を大きく左右するということです。ソフトウェア設計に関する本はたくさんありますが、残念ながら学校の授業でもデバッグについて紹介しているものはほとんどありません。

デバッガーが年々改良されていくにつれて、プログラマーのコーディングスタイルも大きく変わっていきます。もちろん、デバッガーは優れた思考に取って代わることはできませんし、思考は優れたデバッガーに取って代わることはできませんが、最も完璧な組み合わせは、優れたデバッガーと優れた思考です。

次のグラフは、書籍「デバッグ」で紹介されている9つのデバッグルールです。The 9 Indispensable Rules for Finding Even the Most Elusive Software and Hardware Problems＞に記載されている9つのデバッグルールです。

<p align="center">

<img src="Images/debug_rules_en.jpeg" width="500" />

</p>

## アセンブリ言語

> iOSプログラマーとして、仕事のほとんどの時間はアセンブリ言語を扱うことはありませんが、アセンブリを理解することは、特にソースコードのないシステムフレームワークやサードパーティのフレームワークをデバッグする際に、非常に役に立ちます。

アセンブリ言語とは、低レベルの機械指向プログラミング言語であり、様々なCPU用の機械語命令のニーモニックの集合体と考えることができる。また、アセンブリ言語で書かれたプログラムは、実行速度が速い、メモリ使用量が少ないなどのメリットがあります。

現在、アップル社のプラットフォームでは、x86とARMという2つの主要なアーキテクチャが広く使われている。モバイル機器ではARMのアセンブリ言語を使用していますが、これは主にARMが低消費電力の利点を持つ縮小命令セットコンピューティング（RISC）アーキテクチャであるためです。一方、Mac OSなどのデスクトッププラットフォームでは、x86アーキテクチャが使用されています。iOSシミュレータにインストールされたアプリは、実際にはシミュレータ内でMac OSアプリとして動作しており、シミュレータがコンテナのように機能していることになります。今回のケースはiOSシミュレータでデバッグを行ったため、主な研究対象は**x86**アセンブリ言語です。

### AT&T and Intel

x86のアセンブリ言語は、2つの構文に分かれています。インテル(x86プラットフォームのドキュメントで使われていたもの)とAT&Tです。インテルはMS-DOSやWindows系で主流であり、AT&TはUNIX系で一般的です。インテルとAT&Tでは、変数、定数、レジスタへのアクセス、間接アドレス、オフセットなどの構文に大きな違いがあります。このように構文には大きな違いがありますが、ハードウェアシステムは同じなので、どちらか一方をシームレスに移行することができます。XcodeではAT&Tのアセンブリ言語を使用しているので、以下では**AT&T**に焦点を当てて説明します。

> Hopperの逆アセンブルツールではインテルの構文が使われていることに注意してください。
逆アセンブルツールであるHopper DisassembleとIDA Proではインテルの構文が使われています。

インテルとAT&Tの違いは以下の通りです。
1. オペランドの接頭辞。AT&Tの構文では、レジスタ名の接頭辞として`%`、即値オペランドの接頭辞として`$`が使われていますが、インテルではレジスタ、即値オペランドともに接頭辞は使われていません。また、AT&Tでは16進数の接頭辞として「0x」が追加されている点も異なる。下の図は、それぞれの接頭辞の違いを示したものです。

	| AT&T | Intel |
	|:-------:|:-------:|
	| movq %rax, %rbx | mov rbx, rax | | addq $0x10, %rbx
	| addq $0x10, %rsp | add rsp, 010h |

> インテルの構文では、16進数のオペランドには接尾辞「h」が、2進数のオペランドには接尾辞「b」が使われます。

2. オペランド AT&Tのシンタックスでは、第1オペランドがソースオペランド、第2オペランドがデスティネーションオペランドとなります。しかし、インテルの構文では、オペランドの順番が逆になります。この点からも、私たちの読書習慣からすると、AT&Tの構文の方がしっくりきます。
3. アドレッシングモード。AT&Tの間接アドレッシングモードは、インテルの構文と比較して読みにくい。しかし、アドレス計算のアルゴリズムは同じで、「address = disp + base + index * scale」となっています。`base` はベースアドレスを表し、`disp` はオフセットアドレスを表し、`index * scale` は要素の位置を決定し、`scale` は要素のサイズで、2の累乗にしかなりません。`segreg: disp(base,index,scale)` はAT&Tの命令で、 `segreg: [base+index*scale+disp]` はインテルの命令です。実は、上記2つの命令はどちらもセグメントアドレッシングモードに属しています。segreg`はセグメントレジスタの略で、通常、リアルモードではCPUのアドレッシングの桁数がレジスタの桁数を超える場合に使われます。例えば、CPUは20ビットの空間をアドレス指定できますが、レジスタは16ビットしかありません。20桁のスペースを確保するには、別のアドレッシングモードを使用する必要があります。それが `segreg:offset` `です。このアドレッシングモードでは、オフセットアドレスは `segreg * 16 + offset` `となりますが、フラットメモリモードよりも複雑になります。プロテクトモードでは、アドレッシングはリニアアドレス空間の下で行われるので、セグメントベースアドレスは無視できます。

	| AT&T | Intel |
	|:-------:|:-------:|
	| movq 0xb57751(%rip), %rsi | mov rsi, qword ptr [rip+0xb57751h] |
	| leaq (%rax,%rbx,8), %rdi | lea rdi, qword ptr [rax+rbx*8] |
	
> 即値オペランドが `disp` または `scale` の場所に来る場合は、`$` サフィックスは省略できます。インテルの構文では、メモリオペランドの前に、`byte ptr`、`word ptr`、`dword ptr`、`qword ptr`を付ける必要があります。

4. オペコードのサフィックス。AT&Tの構文では、すべてのオペコードにサイズを指定するサフィックスがついています。一般的には、`b`、`w`、`l`、`q` の4種類の接尾辞があります。 `b` は8ビットのバイトを、`w` は16ビットのワードを、`l` は32ビットのダブルワードを表す。32桁のワードは、16ビット時代のロングワードとも呼ばれる。`q` は64ビットのクワッドワードを表す。下の図は、AT&Tとインテルのデータ移行命令（mov）の構文を示しています。

	| AT&T｜Intel |
	|:-------:|:-------:|
	| movb %al, %bl | mov bl, al |
	| movw %ax, %bx｜mov bx, ax｜｜｜
	| movl %eax, %ebx | mov ebx, eax | 
	| movq %rax, %rbx｜mov rbx, rax｜｜｜

### Register

ご存知のように、メモリはCPUの命令やデータを格納するためのものです。メモリは基本的にバイトの配列です。メモリのアクセス速度は非常に高速ですが、CPUの命令実行を高速化するためには、より小型で高速な記憶装置が必要です。それがレジスタです。命令の実行中、すべてのデータは一時的にレジスタに格納されます。これが、レジスタの名前の由来です。

プロセッサが16ビットから32ビットになると、8本のレジスタも32ビットに拡張されます。その後、拡張されたレジスタを使用する際には、元のレジスタ名に「E」という接頭語が追加される。32ビットプロセッサは、Intel Architecture 32ビット、すなわちIA32である。現在、主要なプロセッサは、IA32を拡張した64ビットのインテル・アーキテクチャーで、x86-64と呼ばれています。IA32は過去のものなので、本稿ではx86-64のみを取り上げる。なお、x86-64では、レジスタの数が8本から16本に拡張されています。この拡張のおかげで、プログラムの状態はレジスタに格納できますが、スタックには格納できません。そのため、メモリアクセスの頻度が大幅に減ります。

x86-64では、64ビットの一般レジスターが16本、フローティングポインタレジスターが16本。また、CPUには「rip」と呼ばれる64ビットの命令ポインタ・レジスタがもう1つある。これは、次に実行される命令のアドレスを格納するためのものである。この他にも、あまり使われていないレジスタがいくつかありますが、ここでは触れません。16本のジェネラルレジスタのうち、8本はIA32からのもので、rax、rcx、rdx、rbx、rsi、rdi、rsp、rbpです。残りの8本はx86-64から新たに追加されたジェネラルレジスタで、r8～r15である。16本のフローティングレジスタはxmm0～xmm15です。

現在のCPUは8088からなので、レジスタも16ビットから32ビット、そして64ビットへと拡張されている。そのため、プログラムはレジスタの下位8ビット、16ビット、32ビットにアクセスすることができます。

下の図は、x86-64の16本の一般的なレジスタを示しています。

<p align="center">

<img src="Images/general_register_en.png" height=700 />

</p>

LLDBの`register read`コマンドを使うと、現在のスタックフレームのレジスタデータをダンプすることができます。

例えば、以下のコマンドを使用すると、レジスタ内のすべてのデータを表示することができます。

```
register read -a or register read --all
```

```
General Purpose Registers:
       rax = 0x00007ff8b680c8c0
       rbx = 0x00007ff8b456fe30
       rcx = 0x00007ff8b6804330
       rdx = 0x00007ff8b6804330
       rdi = 0x00007ff8b456fe30
       rsi = 0x000000010cba6309  "initWithTask:delegate:delegateQueue:"
       rbp = 0x000070000f1bcc90
       rsp = 0x000070000f1bcc18
        r8 = 0x00007ff8b680c8c0
        r9 = 0x00000000ffff0000
       r10 = 0x00e6f00100e6f080
       r11 = 0x000000010ca13306  CFNetwork`-[__NSCFURLLocalSessionConnection initWithTask:delegate:delegateQueue:]
       r12 = 0x00007ff8b4687c70
       r13 = 0x000000010a051800  libobjc.A.dylib`objc_msgSend
       r14 = 0x00007ff8b4433bd0
       r15 = 0x00007ff8b6804330
       rip = 0x000000010ca13306  CFNetwork`-[__NSCFURLLocalSessionConnection initWithTask:delegate:delegateQueue:]
    rflags = 0x0000000000000246
        cs = 0x000000000000002b
        fs = 0x0000000000000000
        gs = 0x0000000000000000
       eax = 0xb680c8c0
       ebx = 0xb456fe30
       ecx = 0xb6804330
       edx = 0xb6804330
       edi = 0xb456fe30
       esi = 0x0cba6309
       ebp = 0x0f1bcc90
       esp = 0x0f1bcc18
       r8d = 0xb680c8c0
       r9d = 0xffff0000
      r10d = 0x00e6f080
      r11d = 0x0ca13306
      r12d = 0xb4687c70
      r13d = 0x0a051800
      r14d = 0xb4433bd0
      r15d = 0xb6804330
        ax = 0xc8c0
        bx = 0xfe30
        cx = 0x4330
        dx = 0x4330
        di = 0xfe30
        si = 0x6309
        bp = 0xcc90
        sp = 0xcc18
       r8w = 0xc8c0
       r9w = 0x0000
      r10w = 0xf080
      r11w = 0x3306
      r12w = 0x7c70
      r13w = 0x1800
      r14w = 0x3bd0
      r15w = 0x4330
        ah = 0xc8
        bh = 0xfe
        ch = 0x43
        dh = 0x43
        al = 0xc0
        bl = 0x30
        cl = 0x30
        dl = 0x30
       dil = 0x30
       sil = 0x09
       bpl = 0x90
       spl = 0x18
       r8l = 0xc0
       r9l = 0x00
      r10l = 0x80
      r11l = 0x06
      r12l = 0x70
      r13l = 0x00
      r14l = 0xd0
      r15l = 0x30

Floating Point Registers:
     fctrl = 0x037f
     fstat = 0x0000
      ftag = 0x00
       fop = 0x0000
     fioff = 0x00000000
     fiseg = 0x0000
     fooff = 0x00000000
     foseg = 0x0000
     mxcsr = 0x00001fa1
  mxcsrmask = 0x0000ffff
     stmm0 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0xff 0xff}
     stmm1 = {0x00 0x01 0x00 0x00 0x00 0x00 0x00 0x00 0xff 0xff}
     stmm2 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     stmm3 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     stmm4 = {0x00 0x00 0x00 0x00 0x00 0x00 0xbc 0x87 0x0b 0xc0}
     stmm5 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     stmm6 = {0x00 0x00 0x00 0x00 0x00 0x00 0x78 0xbb 0x0b 0x40}
     stmm7 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm0 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm1 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm2 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm3 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm4 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm5 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm6 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm7 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm8 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      ymm9 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     ymm10 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     ymm11 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     ymm12 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     ymm13 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     ymm14 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     ymm15 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm0 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm1 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm2 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm3 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm4 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm5 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm6 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm7 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm8 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
      xmm9 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     xmm10 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     xmm11 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     xmm12 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     xmm13 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     xmm14 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}
     xmm15 = {0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00}

Exception State Registers:
    trapno = 0x00000003
       err = 0x00000000
  faultvaddr = 0x000000010bb91000  
```

ご存知のように、x86-64にはxmm0～xmm15の16本のフローティング・ポインタ・レジスタがあります。実は、それ以外にもいくつかの詳細がある。register read -a`コマンドの出力を見ると、xmmレジスタ群の他にstmmとymmのレジスタがあることに気づくでしょう。ここで、stmmはstレジスタの別名で、stはx86のFPU（Float Point Unit）でフロートデータを扱うためのレジスタである。FPUには1つのフロート・ポインタ・レジスタがあり、そのレジスタには80ビットのフロート・ポインタ・レジスタが8つ（st0～st7）あります。xmmは128ビットのレジスタで、ymmはxmmを拡張した256ビットのレジスタです。実際には、xmmレジスタはymmレジスタの下位128ビットです。eaxレジスタがraxレジスタの下位32ビットであるように。Pentium IIIでは、インテルは[MMX](https://zh.wikipedia.org/wiki/MMX)を拡張したSSE(Streaming SIMD Extensions)という命令セットを発表した。SSEでは、新たに8つの128ビットレジスタ（xmm0～xmm7）が追加された。AVX(Advanced Vector Extensions)命令セットはSSEの拡張アーキテクチャです。また、AVXでは128ビットのレジスタxmmが256ビットのレジスタymmに拡張された。

<p align="center">

<img src="Images/float_register_en.png" />。

</p>

### Function

関数呼び出しには、パラメータの受け渡しや、あるコンパイルユニットから別のコンパイルユニットへの制御の受け渡しが含まれます。関数呼び出しの手順では、データの受け渡しやローカル変数の割り当て、解放などはスタックによって行われます。また、1つの関数呼び出しに割り当てられたスタックをスタックフレームと呼びます。

> OS X x86-64の関数呼び出し規約は、記事で紹介されている規約と同じです。OS X x86-64の関数呼び出し規則は、[System V Application Binary Interface AMD64 Architecture Processor Supplement](http://www.ucw.cz/~hubicka/papers/abi/)に記載されている規則と同じです。そのため、興味のある方は参考にしてください。

#### The Stack Frame

LLDBのデバッグでは、`bt` コマンドを使って、以下のように現在のスレッドのスタックトレースを表示することができる。

```
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x00000001054e09d4 TestDemo`-[ViewController viewDidLoad](self=0x00007fd349558950, _cmd="viewDidLoad") at ViewController.m:18
    frame #1: 0x00000001064a6931 UIKit`-[UIViewController loadViewIfRequired] + 1344
    frame #2: 0x00000001064a6c7d UIKit`-[UIViewController view] + 27
    frame #3: 0x00000001063840c0 UIKit`-[UIWindow addRootViewControllerViewIfPossible] + 61
    // many other frames are ommitted here
```

実際、`bt`コマンドはスタックフレームの上で動作します。スタックフレームは、関数のリターンアドレスやローカル変数を保持しており、これは関数実行のコンテキストと見ることができます。ご存知のように、ヒープは上に向かって成長し、スタックは下に向かって成長します。つまり、大きな数のメモリアドレスから小さな数のメモリアドレスへと成長します。関数が呼び出されると、その関数を呼び出すために1つの独立したスタックフレームが割り当てられます。フレームポインタと呼ばれるrbpレジスタは、常に割り当てられた最新のスタックフレームの終端（上位アドレス）を指しています。スタックポインタと呼ばれるrspレジスタは，常に割り当てられた最新のスタックフレームの先頭（ローアドレス）を指しています。以下にフレームスタックの図を示します。

<p align="center">

<img src="Images/stack_frame.png" />。

</p>

左側の列「Position」は、間接アドレス方式のメモリアドレスです。Content」は「Position」が指し示すアドレスの値です。上の図のスタックフレームの構造によると、関数呼び出しの手順は、以下のようないくつかのステップに分けて説明できます。
1. 1. 呼び出した関数は，パラメータをスタックにプッシュします．パラメータがない場合，このステップは省略できます．
2. 2. 関数呼び出し後の最初の命令をスタックにプッシュします。これは実際にはリターンアドレスです。
3. 呼び出された関数の開始アドレスにジャンプして実行します。
4. 呼び出された関数は、%rbp レジスタに開始アドレスを保持します。
5. 5. %rspレジスタの値を%rbpレジスタに保存し、%rbpレジスタが呼び出された関数のスタックフレームの開始アドレスを指すようにする。
6. 6. 呼び出された関数のレジスタをスタックにプッシュします。これはオプションである．

ステップ2とステップ3は、実際には`call`命令に属します。また、ステップ4とステップ5は、アセンブリ命令では以下のように記述できます。

```
TestDemo`-[ViewController viewDidLoad]:
    0x1054e09c0 <+0>:  pushq  %rbp //step 4
    0x1054e09c1 <+1>:  movq   %rsp, %rbp //step 5
```

この2つのステップは、それぞれの関数を呼び出す際に行われていることがわかります。上の図にはもう一つ詳細があります。rspレジスタの下に赤い領域がありますが、これはABIでレッドゾーンと呼ばれています。これは予約領域であり、シグナルハンドラや割り込みハンドラで変更してはいけません。したがって、リーフ関数、つまり、他の関数を呼び出さない関数は、この領域を一時的なデータ用に使用することができます。

```
UIKit`-[UIViewController loadViewIfRequired]:
    0x1064a63f1 <+0>:    pushq  %rbp
    0x1064a63f2 <+1>:    movq   %rsp, %rbp
    0x1064a63f5 <+4>:    pushq  %r15
    0x1064a63f7 <+6>:    pushq  %r14
    0x1064a63f9 <+8>:    pushq  %r13
    0x1064a63fb <+10>:   pushq  %r12
    0x1064a63fd <+12>:   pushq  %rbx
```

上記の命令のうち、`0x1064a63f5`から`0x1064a63fd`までの命令はステップ6に属します。レジスタには関数保持レジスタと呼ばれるものがあり、呼び出した関数に属するが、呼び出された関数がその値を保持する必要があることを意味します。下のアセンブリ命令から、rbx、rsp、r12～r15がこのようなレジスタに属していることがわかります。

```
    0x1064a6c4b <+2138>: addq   $0x1f8, %rsp              ; imm = 0x1F8 
    0x1064a6c52 <+2145>: popq   %rbx
    0x1064a6c53 <+2146>: popq   %r12
    0x1064a6c55 <+2148>: popq   %r13
    0x1064a6c57 <+2150>: popq   %r14
    0x1064a6c59 <+2152>: popq   %r15
    0x1064a6c5b <+2154>: popq   %rbp
    0x1064a6c5c <+2155>: retq   
    0x1064a6c5d <+2156>: callq  0x106d69e9c               ; symbol stub for: __stack_chk_fail
```

#### Call instruction

関数を呼び出すための命令は `call` で、以下を参照してください。

```
call function
```

パラメータの `function` は **TEXT** セグメントのプロシージャです。`call` 命令は2つのステップに分けられます。最初のステップは、`call` 命令の次の命令のアドレスをスタックにプッシュすることです。ここで、次のアドレスは、実際には、呼び出された関数が終了した後のリターンアドレスです。2つ目のステップは `function` へのジャンプです。call`命令は、以下の2つの命令に相当します。

```
push next_instruction
jmp  function
```

以下は、iOSシミュレータでの`call`命令の例です。

```
    0x10915c714 <+68>: callq 0x1093ca502 ; symbol stub for: objc_msgSend
    0x105206433 <+66>: callq *0xb3cd47(%rip) ; (void *)0x000000010475e800: objc_msgSend
```

上のコードは、`call` 命令の2つの使い方を示しています。1つ目の使い方では、オペランドはメモリアドレスで、実際にはMach-Oファイルのシンボルスタブです。これにより、動的リンカーを使って関数のシンボルを検索することができます。2番目の使い方では、オペランドは実際には間接アドレッシングモードで取得されます。なお、AT&Tの構文では、ジャンプ／コール命令（またはプログラマカウンタ関連のジャンプ）の即値オペランドには、プレフィックスとして`*`を付ける必要があります。

#### Ret instruction

一般に，`ret`命令は，呼び出された関数から呼び出した関数にプロシージャを戻すために使用されます。この命令は、スタックの先頭からアドレスをポップし、そのアドレスにジャンプバックして実行を続けます。上の例では、`next_instruction`にジャンプバックしています。ret`命令が実行される前に、呼び出した関数に属するレジスタがポップされます。これは、関数呼び出し手順のステップ6ですでに述べたとおりです。

#### Parameter passing and return value

ほとんどの関数には、整数、浮動小数点、ポインタなどのパラメータがあります。また、関数には通常、実行結果が成功したか失敗したかを示す戻り値があります。OSXでは、最大6個のパラメータを、rdi、rsi、rdx、rcx、r8、r9の順にレジスタを介して渡すことができます。では、6個以上のパラメータを持つ関数はどうでしょうか？もちろん、このような状況もあります。このような場合、スタックを使って残りのパラメータを逆の順序で保存することができます。OSXには8つの浮動小数点レジスタがあり、最大8つの浮動小数点パラメータを渡すことができます。

関数の戻り値については、整数の戻り値を保存するために `rax` レジスタが使われます。戻り値が浮動小数点の場合は、xmm0～xmm1レジスタを使用する。下の図は、関数呼び出し時のレジスタ使用規則を明確に示しています。

<p align="center">

<img src="Images/register_usage.png" />。

</p>

Preserved across function calls`は、レジスタが関数呼び出しの間も保存される必要があるかどうかを示します。前述のrbx、r12～r15レジスタの他に、rspとrbpレジスタもcalle-savedレジスタに属していることがわかります。これは、この2つのレジスタが、プログラムスタックを指す重要なロケーションポインターを予約しているからです。

次に、実際の例に沿って、関数呼び出しの命令を説明します。例えば，`CocoaLumberjack`のマクロ`DDLogError`を例にとります．このマクロが呼び出されると、クラスメソッド `log:level:flag:context:file:function:line:tag:format:` が呼び出されます。以下のコードと説明は `DDLogError` の呼び出しとそれに対応するアセンブリ命令についてです。

```
- (IBAction)test:(id)sender {
    DDLogError(@"TestDDLog:%@", sender);
}
```

```
    0x102c568a3 <+99>:  xorl   %edx, %edx
    0x102c568a5 <+101>: movl   $0x1, %eax
    0x102c568aa <+106>: movl   %eax, %r8d
    0x102c568ad <+109>: xorl   %eax, %eax
    0x102c568af <+111>: movl   %eax, %r9d
    0x102c568b2 <+114>: leaq   0x2a016(%rip), %rcx       ; "/Users/dev-aozhimin/Desktop/TestDDLog/TestDDLog/ViewController.m"
    0x102c568b9 <+121>: leaq   0x2a050(%rip), %rsi       ; "-[ViewController test:]"
    0x102c568c0 <+128>: movl   $0x22, %eax
    0x102c568c5 <+133>: movl   %eax, %edi
    0x102c568c7 <+135>: leaq   0x2dce2(%rip), %r10       ; @"\eTestDDLog:%@"
    0x102c568ce <+142>: movq   0x33adb(%rip), %r11       ; (void *)0x0000000102c8ad18: DDLog
    0x102c568d5 <+149>: movq   0x34694(%rip), %rbx       ; ddLogLevel
    0x102c568dc <+156>: movq   -0x30(%rbp), %r14
    0x102c568e0 <+160>: movq   0x332f9(%rip), %r15       ; "log:level:flag:context:file:function:line:tag:format:"
    0x102c568e7 <+167>: movq   %rdi, -0x48(%rbp)
    0x102c568eb <+171>: movq   %r11, %rdi
    0x102c568ee <+174>: movq   %rsi, -0x50(%rbp)
    0x102c568f2 <+178>: movq   %r15, %rsi
    0x102c568f5 <+181>: movq   %rcx, -0x58(%rbp)
    0x102c568f9 <+185>: movq   %rbx, %rcx
    0x102c568fc <+188>: movq   -0x58(%rbp), %r11
    0x102c56900 <+192>: movq   %r11, (%rsp)
    0x102c56904 <+196>: movq   -0x50(%rbp), %rbx
    0x102c56908 <+200>: movq   %rbx, 0x8(%rsp)
    0x102c5690d <+205>: movq   $0x22, 0x10(%rsp)
    0x102c56916 <+214>: movq   $0x0, 0x18(%rsp)
    0x102c5691f <+223>: movq   %r10, 0x20(%rsp)
    0x102c56924 <+228>: movq   %r14, 0x28(%rsp)
    0x102c56929 <+233>: movb   $0x0, %al
    0x102c5692b <+235>: callq  0x102c7d2be               ; symbol stub for: objc_msgSend
```

Objective-Cのすべての関数は、`objc_msgSend`関数の呼び出しになるので、`log:level:flag:context:file:function:line:tag:format:`メソッドは最終的に以下のコードになります。

```
objc_msgSend(DDLog, @selector(log:level:flag:context:file:function:line:tag:format:), asynchronous, level, flag, context, file, function, line, tag, format, sender)
```

パラメータの受け渡しに使用できるレジスタの数は最大6個であることはすでに述べました。余ったパラメータはスタックを使って受け渡しを行うことができます。上記の関数には6個以上のパラメータがあるので、パラメータの受け渡しにはレジスタとスタックの両方が使われます。以下の2つの表は、`DDLogError`関数を呼び出す際の、レジスタとスタックの詳細な使用方法を示しています。

|  一般的なレジスタ | 値| パラメータ | アセンブリ命令 | コメント|
|:-------:|:-------:|:-------:|:-------:|:-------:|
| rdi | DDLog | self | 0x102c568eb <+171>: movq %r11, %rdi | |
| rsi | "log:level:flag:context:file:function:line:tag:format:" | op｜0x102c568f2 <+178>: movq %r15, %rsi| |
| 0x102c568a3 <+99>: xorl %edx, %edx | xorlは、排他的論理和演算です。ここでは edx レジスタのクリアに使用しています。 |
| rcx | 18446744073709551615 | level | 0x102c568f9 <+185>: movq %rbx, %rcx | (DDLogLevelAll or NSUIntegerMax) |
| 0x102c568aa <+106>: movl %eax, %r8d｜DDLogFlagError｜（DDLogLevelAllまたはNSIntegerMax |
| r9 | 0 | context | 0x102c568af <+111>: movl %eax, %r9d | | DDLogFlagError |

| スタックフレームオフセット|値|パラメータ|アセンブリ命令|コメント|
|:-------:|:-------:|:-------:|:-------:|:-------:|
| (%rsp) | "/Users/dev-aozhimin/Desktop/TestDDLog/TestDDLog/ViewController.m" | ファイル | 0x102c56900 <+192>: movq %r11, (%rsp) | |
| 0x8(%rsp) | "-[ViewController test:]" | ファンクション | 0x102c56908 <+200>: movq %rbx, 0x8(%rsp) | |
| 0x10(%rsp) | 0X22 | 行 | 0x102c5690d <+205>: movq $0x22, 0x10(%rsp) | 対応するDDLogErrorの起動は34行目にあります | 
| 0x18(%rsp) | 0X0 | タグ | 0x102c56916 <+214>: movq $0x0, 0x18(%rsp) | nil |
| 0x20(%rsp) | "TestDDLog:%@" | フォーマット | 0x102c5691f <+223>: movq %r10, 0x20(%rsp) | |
| 0x28(%rsp) | UIButtonのインスタンス | 可変パラメータの第1パラメータ | 0x102c56924 <+228>: movq %r14, 0x28(%rsp) | UIButtonのインスタンス |

他にも，`po $rsi`を使って整数形式の値を印刷することもできます．

アセンブリ言語の助けを借りれば、デバッグ時に非常に必要となる低レベルの知識を調べることができます。アセンブリ関連の知識をできるだけ詳しく紹介しようと努力しています。しかし、アセンブリの知識階層は膨大であり、1回の記事では説明しきれません。前述の文献を参考にしてください。また、**CSAPP**の第3章 -- Machine level representation of a programもお勧めです。これは稀に見る良い参考資料だと思います。

## Case

この記事では、実際のケースを通して、デバッグの手順を説明しています。個人情報保護のため、一部内容を変更しています。

### Issue

今回ご紹介する問題は、私がログインSDKを開発していたときに起こったものです。あるユーザーが、ログインページで「QQ」ボタンを押すとアプリがクラッシュすると言っていました。この問題をデバッグしたところ、QQアプリが同時にインストールされていないとクラッシュが起こることがわかりました。ユーザーがQQボタンを押してログインを要求すると、QQログインSDKはアプリ内の認証Webページを起動しようとします。この場合、認識されないセレクタエラー `[TCWebViewController setRequestURLStr:]` が発生します。

> P.S: 問題に集中するため、不要なビジネスデバッグ情報は以下に記載していません。また、アプリ名には **AADebug** を使用しています。

以下は、このクラッシュのスタックトレースです。

```
Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[TCWebViewController setRequestURLStr:]: unrecognized selector sent to instance 0x7fe25bd84f90'
*** First throw call stack:
(
	0   CoreFoundation                      0x0000000112ce4f65 __exceptionPreprocess + 165
	1   libobjc.A.dylib                     0x00000001125f7deb objc_exception_throw + 48
	2   CoreFoundation                      0x0000000112ced58d -[NSObject(NSObject) doesNotRecognizeSelector:] + 205
	3   AADebug                             0x0000000108cffefc __ASPECTS_ARE_BEING_CALLED__ + 6172
	4   CoreFoundation                      0x0000000112c3ad97 ___forwarding___ + 487
	5   CoreFoundation                      0x0000000112c3ab28 _CF_forwarding_prep_0 + 120
	6   AADebug                             0x000000010a663100 -[TCWebViewKit open] + 387
	7   AADebug                             0x000000010a6608d0 -[TCLoginViewKit loadReqURL:webTitle:delegate:] + 175
	8   AADebug                             0x000000010a660810 -[TCLoginViewKit openWithExtraParams:] + 729
	9   AADebug                             0x000000010a66c45e -[TencentOAuth authorizeWithTencentAppAuthInSafari:permissions:andExtraParams:delegate:] + 701
	10  AADebug                             0x000000010a66d433 -[TencentOAuth authorizeWithPermissions:andExtraParams:delegate:inSafari:] + 564
………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………

Lines of irrelevant information are removed here

………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………
236
14  libdispatch.dylib                   0x0000000113e28ef9 _dispatch_call_block_and_release + 12
	15  libdispatch.dylib                   0x0000000113e4949b _dispatch_client_callout + 8
	16  libdispatch.dylib                   0x0000000113e3134b _dispatch_main_queue_callback_4CF + 1738
	17  CoreFoundation                      0x0000000112c453e9 __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__ + 9
	18  CoreFoundation                      0x0000000112c06939 __CFRunLoopRun + 2073
	19  CoreFoundation                      0x0000000112c05e98 CFRunLoopRunSpecific + 488
	20  GraphicsServices                    0x0000000114a13ad2 GSEventRunModal + 161
	21  UIKit                               0x0000000110d3f676 UIApplicationMain + 171
	22  AADebug                             0x0000000108596d3f main + 111
	23  libdyld.dylib                       0x0000000113e7d92d start + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```

### Message Forwarding

デバッグの話をする前に、Objective-C のメッセージフォワーディングについて知っておきましょう。ご存知のように、Objective-Cは関数呼び出しではなく、メッセージング構造を使用します。重要な違いは、メッセージング構造では、どの関数を実行するかはコンパイル時ではなくランタイムが決定するということです。つまり、あるオブジェクトに認識されないメッセージが送られても、コンパイル時には何も起こりません。また、ランタイム中に、理解できないメソッドを受け取った場合、オブジェクトはメッセージフォワーディングを行います。これは、開発者として未知のメッセージをどのように処理するかをメッセージに伝えることができるように設計されたプロセスです。

メッセージフォワーディングの際には、通常、以下の4つのメソッドが関係します。

1. `+ (BOOL)resolveInstanceMethod:(SEL)sel`: このメソッドは、未知のメッセージがオブジェクトに渡されたときに呼び出されます。このメソッドは、見つからなかったセレクタを受け取り、そのセレクタを扱えるようになったクラスにインスタンスメソッドが追加されたかどうかを示すブール値を返します。クラスがこのセレクタを処理できる場合は Yes を返し、メッセージの転送処理が完了します。このメソッドは、CoreDataのNSManagedObjectの@dynamicプロパティに動的にアクセスするためによく使われます。`+ (BOOL)resolveClassMethod:(SEL)sel` メソッドは、上記のメソッドと似ていますが、唯一の違いは、このメソッドはクラスメソッドであり、もう一方はインスタンスメソッドであることです。

2. `- (id)forwardingTargetForSelector:(SEL)aSelector`：このメソッドは未知のメッセージを処理するための第2のレシーバを提供し、`forwardInvocation:`よりも高速です。このメソッドは、多重継承の機能を模倣するために使用することができます。ただし、この転送経路の一部を使ってメッセージを操作する方法はありません。置換された受信者に送信する前にメッセージを変更する必要がある場合は、完全な転送メカニズムを使用する必要があります。

3. `- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`：フォワーディングアルゴリズムがここまで来たら、完全なフォワーディングメカニズムを開始する。NSMethodSignature`は、aSelectorパラメータにメソッドの説明を含むこのメソッドによって返されます。なお、メッセージ転送時にセレクタ、ターゲット、引数を含む `NSInvocation` オブジェクトを作成したい場合は、このメソッドをオーバーライドする必要があります。

4. `- (void)forwardInvocation:(NSInvocation *)anInvocation`：このメソッドの実装には以下の部分が含まれます。そのオブジェクトにメッセージを送り，anInvocationはその戻り値を保存し，ランタイムはその戻り値を元のメッセージ送信者に送ります．実はこのメソッドは，呼び出し先を変更してから呼び出すだけで，`ForwardingTargetForSelector:`メソッドと同じ動作をすることができますが，ほとんど行いません．

通常、メッセージ転送に使われる最初の2つのメソッドは、**高速転送**と呼ばれます。高速転送と区別するために、3番目と4番目の方法を **通常の転送** または **通常の転送** と呼びます。メッセージの転送を完了するために**NSInvocation**オブジェクトを作成する必要があるため、非常に低速です。

> 注意: `methodSignatureForSelector` メソッドがオーバーライドされていない場合や、返される `NSMethodSignature` が nil の場合、`forwardInvocation` は呼び出されず、`doesNotRecognizeSelector` エラーが発生してメッセージ転送が終了します。これは、以下の `__forwarding__` 関数のソースコードから見ることができます。

メッセージ転送のプロセスは、以下のようなフロー図で説明することができます。

<p align="center">

<img src="Images/message_forward_en.png" />

</p>

フロー図で説明したように、各ステップでは、受信者にメッセージを処理する機会が与えられます。各ステップは、その前のステップよりもコストがかかります。ベストプラクティスは、できるだけ早い段階でメッセージの転送処理を行うことである。メッセージがすべてのプロセスで処理されなかった場合は、`doesNotRecognizeSeletor`エラーが発生して、セレクタがオブジェクトに認識されないことを示します。

### Debugging Process

そろそろ理論の部分を終えて問題に戻ります。

トレーススタックからの`TCWebViewController`の情報によると、当然Tencent SDKの**TencentOpenAPI.framework**に関連していますが、最近Tencent SDKをアップデートしていないので、クラッシュは**TencentOpenAPI.framework**が原因ではないことになります。

まず、コードを逆コンパイルして、`TCWebViewController`クラスの構造体を取得しました。

```
@class TCWebViewController : UIViewController<UIWebViewDelegate, NSURLConnectionDelegate, NSURLConnectionDataDelegate> {
    @property webview
    @property webTitle
    @property requestURLStr
    @property error
    @property delegate
    @property activityIndicatorView
    @property finished
    @property theData
    @property retryCount
    @property hash
    @property superclass
    @property description
    @property debugDescription
    ivar _nloadCount
    ivar _webview
    ivar _webTitle
    ivar _requestURLStr
    ivar _error
    ivar _delegate
    ivar _xo
    ivar _activityIndicatorView
    ivar _finished
    ivar _theData
    ivar _retryCount
    -setError:
    -initWithNibName:bundle:
    -dealloc
    -stopLoad
    -doClose
    -viewDidLoad
    -loadReqURL
    -viewDidDisappear:
    -shouldAutorotateToInterfaceOrientation:
    -supportedInterfaceOrientations
    -shouldAutorotate
    -webViewDidStartLoad:
    -webViewDidFinishLoad:
    -webView:didFailLoadWithError:
    -webView:shouldStartLoadWithRequest:navigationType:
}
```

静的解析の結果、`TCWebViewController` には `requestURLStr` の Setter と Getter メソッドがありませんでした。それは、`TCWebViewController`のプロパティが、**Core Data**フレームワークのように、実行時に動的に生成されるのではなく、`@dynamic`を使ってコンパイラにプロパティのゲッターとセッターを生成するように指示する動的な方法で実装されているのではないか、というものでした。そこで、私たちの推測が正しいかどうかを確かめるために、このアイデアを深く追求することにしました。追跡中に、**TencentOpenAPI.framework**内の`NSObject`に対して、`NSObject(MethodSwizzlingCategory)`というカテゴリーがあり、これが非常に怪しいことがわかりました。このカテゴリの中には、`switchMethodForCodeZipper`というメソッドがあり、その実装は`QQmethodSignatureForSelector`と`QQforwardInvocation`の`methodSignatureForSelector`と`QQforwardInvocation`のメソッドを置き換えていました。

```objective-c
void +[NSObject switchMethodForCodeZipper](void * self, void * _cmd) {
    rbx = self;
    objc_sync_enter(self);
    if (*(int8_t *)_g_instance == 0x0) {
            [NSObject swizzleMethod:@selector(methodSignatureForSelector:) withMethod:@selector(QQmethodSignatureForSelector:)];
            [NSObject swizzleMethod:@selector(forwardInvocation:) withMethod:@selector(QQforwardInvocation:)];
            *(int8_t *)_g_instance = 0x1;
    }
    rdi = rbx;
    objc_sync_exit(rdi);
    return;
}
```

Then we kept tracking into `QQmethodSignatureForSelector` method, and there was a method named `_AddDynamicPropertysSetterAndGetter` in it. From the name, we can easily get that this method is to add Setter and Getter method for properties dynamically. This found can substantially verify our original guess is correct. 

```objective-c
void * -[NSObject QQmethodSignatureForSelector:](void * self, void * _cmd, void * arg2) {
    r14 = arg2;
    rbx = self;
    rax = [self QQmethodSignatureForSelector:rdx];
    if (rax == 0x0) {
            rax = sel_getName(r14);
            _AddDynamicPropertysSetterAndGetter();
            rax = 0x0;
            if (0x0 != 0x0) {
                    rax = [rbx methodSignatureForSelector:r14];
            }
    }
    return rax;
}
```

しかし、なぜ`TCWebViewController`クラスではセッターが認識できないのか？このバージョンの開発中に `QQMethodSignatureForSelector` メソッドがカバーされたからでしょうか？しかし、コードの隅々まで調べても手がかりは見つかりませんでした。
非常に残念でした。ここまでで静的解析は終わりました。次のステップは、LLDBを使ってTencent SDKを動的にデバッグし、メッセージフォワーディングプロセスでどのパスがゲッターとセッターの生成を壊したかを調べます。

> LLDBコマンドで`setRequestURLStr`にブレークポイントを設定しようとすると、設定できないことがわかります。その理由は、コンパイル時にセッターが利用できないからです。これは、私たちの最初の推測を検証することができます。

クラッシュのスタックトレースによると、`setRequestURLStr`は` -[TCWebViewKit open]`メソッドの中で呼び出されていると結論付けられます。つまり、Tencent SDKがQQアプリがインストールされているかどうかをチェックし、認証Webページのプログレスを開いている間にクラッシュが起こるということです。

そこで、以下のLLDBコマンドを使用して、このメソッドにブレークポイントを設定します。

```
br s -n "-[TCWebViewKit open]"
```

> `br s` は `breakpoint set` の略で、`-n` はその後ろのメソッド名に応じてブレークポイントを設定することを表し、シンボリックブレークポイントと同じ動作をします。 b -[TCWebViewKit open]` もここでは動作しますが、ここでの `b` は `_regexp-break` の略で、正規表現を使ってブレークポイントを設定します。最後に、`br s -a 0x000000010940b24e`のように、メモリアドレスにブレークポイントを設定することもできます。これは、ブロックのアドレスが利用可能であれば、ブロックのデバッグに役立ちます。

これで、ブレークポイントの設定が完了しました。

```
Breakpoint 34: where = AADebug`-[TCWebViewKit open], address = 0x0000000103157f7d
```

アプリがWeb認証ページを起動しようとすると、このブレークポイントでプロジェクトが停止します。以下を参照してください。

<p align="center">

<img src="Images/lldb_webviewkit_open.png" />。

</p>

iPhoneをお使いの場合、アセンブリコードはARMになります。しかし、解析方法は同じですので、ご注意ください。

96行目にブレークポイントを設定します。このアセンブリコードは、`setRequestURLStr`メソッドの呼び出しであり、`rbx`レジスタの内容を表示すると、`TCWebViewController`インスタンスがこのレジスタに保存されていることがわかります。

#### methodSignatureForSelector

次に LLDB を使って、`QQmethodSignatureForSelector` メソッドにブレークポイントを設定します。

```
br s -n "-[NSObject QQmethodSignatureForSelector:]"

```

LLDBに`c`を入力してブレークポイントを継続させると、`QQmethodSignatureForSelector`メソッドの中でブレークポイントが停止しますので、`QQmethodSignatureForSelector`メソッドが私たちのコードと衝突するという前の推測が無効であることが証明されます。
<p align="center">

<img src="Images/lldb_method_signature.png" />。

</p>

QQmethodSignatureForSelector`メソッドの最後、つまり31行目の`retq`コマンドにブレークポイントを設定します。次に、レジスタ `rax` のメモリアドレスを表示します。

<p align="center">

<img src="Images/lldb_method_signature_1.png" />。

</p>

レジスタ `rax` のメモリアドレス `0x00007fdb36d38df0` をプリントすると、`NSMethodSignature` オブジェクトが返されます。X86アセンブリ言語の設計上の慣習では、戻り値はレジスタ `rax` に保存されます。どうやら `QQmethodSignatureForSelector` メソッドが呼び出されて正しい値を返しているようなので、この問題の追跡を続ける必要があります。

#### forwardInvocation

LLDB経由で`QQforwardInvocation`にブレークポイントを設定します。

```
br s -n "-[NSObject QQforwardInvocation:]"
```

ブレークポイントを設定した後、プログラムの実行を続けるとアプリがクラッシュしてしまいます。そして、`QQforwardInvocation`メソッドはまだ呼ばれていません。これで、`QQforwardInvocation` メソッドが私たちのコードによってコンフリクトしていると結論付けられます。

<p align="center">

<img src="Images/lldb_method_signature_2.png" />。

</p>

`__forwarding___` 関数は、メッセージ転送機構の実装全体を含んでおり、分解コードは[Objective-C 消息发送与转发机制原理](http://yulingtianxia.com/blog/2016/06/15/Objective-C-Message-Sending-and-Forwarding/)から選択しています。この記事では、`forwardingTargetForSelector`メソッドを呼び出す際に、`forwarding`と`receiver`の間で誤っていると思われる判定があります。ここでは、`forwardingTarget`と`receiver`の間で判断する必要があります。以下のコードを参照してください。

```
int __forwarding__(void *frameStackPointer, int isStret) {
  id receiver = *(id *)frameStackPointer;
  SEL sel = *(SEL *)(frameStackPointer + 8);
  const char *selName = sel_getName(sel);
  Class receiverClass = object_getClass(receiver);

  // call forwardingTargetForSelector:
  if (class_respondsToSelector(receiverClass, @selector(forwardingTargetForSelector:))) {
    id forwardingTarget = [receiver forwardingTargetForSelector:sel];
    if (forwardingTarget && forwardingTarget != receiver) {
    	if (isStret == 1) {
    		int ret;
    		objc_msgSend_stret(&ret,forwardingTarget, sel, ...);
    		return ret;
    	}
      return objc_msgSend(forwardingTarget, sel, ...);
    }
  }

  // Zombie Object
  const char *className = class_getName(receiverClass);
  const char *zombiePrefix = "_NSZombie_";
  size_t prefixLen = strlen(zombiePrefix); // 0xa
  if (strncmp(className, zombiePrefix, prefixLen) == 0) {
    CFLog(kCFLogLevelError,
          @"*** -[%s %s]: message sent to deallocated instance %p",
          className + prefixLen,
          selName,
          receiver);
    <breakpoint-interrupt>
  }

  // call methodSignatureForSelector first to get method signature , then call forwardInvocation
  if (class_respondsToSelector(receiverClass, @selector(methodSignatureForSelector:))) {
    NSMethodSignature *methodSignature = [receiver methodSignatureForSelector:sel];
    if (methodSignature) {
      BOOL signatureIsStret = [methodSignature _frameDescriptor]->returnArgInfo.flags.isStruct;
      if (signatureIsStret != isStret) {
        CFLog(kCFLogLevelWarning ,
              @"*** NSForwarding: warning: method signature and compiler disagree on struct-return-edness of '%s'.  Signature thinks it does%s return a struct, and compiler thinks it does%s.",
              selName,
              signatureIsStret ? "" : not,
              isStret ? "" : not);
      }
      if (class_respondsToSelector(receiverClass, @selector(forwardInvocation:))) {
        NSInvocation *invocation = [NSInvocation _invocationWithMethodSignature:methodSignature frame:frameStackPointer];

        [receiver forwardInvocation:invocation];

        void *returnValue = NULL;
        [invocation getReturnValue:&value];
        return returnValue;
      } else {
        CFLog(kCFLogLevelWarning ,
              @"*** NSForwarding: warning: object %p of class '%s' does not implement forwardInvocation: -- dropping message",
              receiver,
              className);
        return 0;
      }
    }
  }

  SEL *registeredSel = sel_getUid(selName);

  // if selector already registered in Runtime
  if (sel != registeredSel) {
    CFLog(kCFLogLevelWarning ,
          @"*** NSForwarding: warning: selector (%p) for message '%s' does not match selector known to Objective C runtime (%p)-- abort",
          sel,
          selName,
          registeredSel);
  } // doesNotRecognizeSelector
  else if (class_respondsToSelector(receiverClass,@selector(doesNotRecognizeSelector:))) {
    [receiver doesNotRecognizeSelector:sel];
  } 
  else {
    CFLog(kCFLogLevelWarning ,
          @"*** NSForwarding: warning: object %p of class '%s' does not implement doesNotRecognizeSelector: -- abort",
          receiver,
          className);
  }

  // The point of no return.
  kill(getpid(), 9);
}
```

基本的には、分解コードを読むことで明確に理解することができます。
 まず、メッセージの転送処理中に `forwardingTargetForSelector` メソッドを呼び出して代替レシーバーを取得します。これは高速転送フェーズとも呼ばれます。forwardingTarget`がnilを返すか、同じレシーバーを返す場合、メッセージの転送は通常の転送フェーズに変わります。基本的には、`methodSignatureForSelector`メソッドを呼び出してメソッドのシグネチャを取得し、それを`frameStackPointer`とともに使用して`invocation`オブジェクトをインスタンス化します。次に `receiver` の `forwardInvocation:` メソッドを呼び出し、引数として先ほどの `invocation` オブジェクトを渡します。結局、`methodSignatureForSelector`メソッドが実装されておらず、`selector`が既にランタイムシステムに登録されている場合は、`doesNotRecognizeSelector:`が呼び出されてエラーになります。

クラッシュのスタックトレースから``forwarding___`を精査すると、メッセージ転送パス全体の中で2番目のパスとして呼び出されていることがわかります。つまり、`forwardInvocation`が呼び出されたときに、`NSInvocation`オブジェクトが呼び出されていることになります。

> また、ブレークポイントの後、ステップごとにコマンドを実行して、アセンブリコードの実行経路を観察しても、同じ結果が得られるはずです。

<p align="center">

<img src="Images/___forwarding___.png" />。

</p>

そして、`forwardInvocation`が呼ばれたときに実行されるのはどのメソッドでしょうか。スタックトレースを見ると、`__ASPECTS_ARE_BEING_CALLED__`という名前のメソッドが実行されていることがわかります。プロジェクト全体のこのメソッドを見渡してみると、最終的には `forwardInvocation` が `Aspects` フレームワークによってフックされていることがわかります。

```objective-c
static void aspect_swizzleForwardInvocation(Class klass) {
    NSCParameterAssert(klass);
    // If there is no method, replace will act like class_addMethod.
    IMP originalImplementation = class_replaceMethod(klass, @selector(forwardInvocation:), (IMP)__ASPECTS_ARE_BEING_CALLED__, "v@:@");
    
    if (originalImplementation) {
        class_addMethod(klass, NSSelectorFromString(AspectsForwardInvocationSelectorName), originalImplementation, "v@:@");
    }
    AspectLog(@"Aspects: %@ is now aspect aware.", NSStringFromClass(klass));
}
```

```objective-c
// This is the swizzled forwardInvocation: method.
static void __ASPECTS_ARE_BEING_CALLED__(__unsafe_unretained NSObject *self, SEL selector, NSInvocation *invocation) {
    NSLog(@"selector:%@",  NSStringFromSelector(invocation.selector));
    NSCParameterAssert(self);
    NSCParameterAssert(invocation);
    SEL originalSelector = invocation.selector;
	SEL aliasSelector = aspect_aliasForSelector(invocation.selector);
    invocation.selector = aliasSelector;
    AspectsContainer *objectContainer = objc_getAssociatedObject(self, aliasSelector);
    AspectsContainer *classContainer = aspect_getContainerForClass(object_getClass(self), aliasSelector);
    AspectInfo *info = [[AspectInfo alloc] initWithInstance:self invocation:invocation];
    NSArray *aspectsToRemove = nil;

    // Before hooks.
    aspect_invoke(classContainer.beforeAspects, info);
    aspect_invoke(objectContainer.beforeAspects, info);

    // Instead hooks.
    BOOL respondsToAlias = YES;
    if (objectContainer.insteadAspects.count || classContainer.insteadAspects.count) {
        aspect_invoke(classContainer.insteadAspects, info);
        aspect_invoke(objectContainer.insteadAspects, info);
    }else {
        Class klass = object_getClass(invocation.target);
        do {
            if ((respondsToAlias = [klass instancesRespondToSelector:aliasSelector])) {
                [invocation invoke];
                break;
            }
        }while (!respondsToAlias && (klass = class_getSuperclass(klass)));
    }

    // After hooks.
    aspect_invoke(classContainer.afterAspects, info);
    aspect_invoke(objectContainer.afterAspects, info);

    // If no hooks are installed, call original implementation (usually to throw an exception)
    if (!respondsToAlias) {
        invocation.selector = originalSelector;
        SEL originalForwardInvocationSEL = NSSelectorFromString(AspectsForwardInvocationSelectorName);
        if ([self respondsToSelector:originalForwardInvocationSEL]) {
            ((void( *)(id, SEL, NSInvocation *))objc_msgSend)(self, originalForwardInvocationSEL, invocation);
        }else {
            [self doesNotRecognizeSelector:invocation.selector];
        }
    }

    // Remove any hooks that are queued for deregistration.
    [aspectsToRemove makeObjectsPerformSelector:@selector(remove)];
}
```

TCWebViewController`はTencent SDKのプライベートクラスなので、他のクラスから直接フックされることはないだろう。しかし、そのスーパークラスがフックされ、このクラスにも影響を与える可能性がある。この推測のもと、私たちは調査を続けました。そして、ようやく答えが見えてきました。UIViewController "をフックしているコードを削除するかコメントすることで、QQ経由でログインしてもアプリがクラッシュしなくなったのです。これまでのところ、クラッシュの原因が `Aspects` フレームワークにあることは間違いありませんでした。

<p align="center">

<img src="Images/answer.png" />。

</p>

doesNotRecognizeSelector:` エラーは、`forwardInvocation:` メソッドの IMP を **Aspects** で置き換えるために使用される `__ASPECTS_ARE_BEING_CALLED__` メソッドによってスローされます。ASPECTS_ARE_BEING_CALLED__`メソッドの実装では、`Aspect`にフックする前、代わりに、後に対応するタイムスライスを持っています。上記のコードのうち、`aliasSelector`は、`aspects__setRequestURLStr:`のように、**Aspects**で処理されるSELです。

Instead hooksの部分では、invocation.targetがaliasSelectorに応答できるかどうかがチェックされます。サブクラスが応答できない場合は、スーパークラスがチェックされ、スーパークラスのスーパークラス、そしてルートクラスまでチェックされます。aliasSelector が応答できないため、respondsToAlias は false です。
respondsToAliasはfalseとなります。そして，originalSelectorが呼び出しのセレクタとして割り当てられます．次に objc_msgSend は，元の SEL を呼び出すために invocation を起動します．TCWebViewControllerは、`originalSelector:setRequestURLStr:`メソッドに応答できないため、最終的にAspectsの**ASPECTS_ARE_BEING_CALLED**メソッドを実行し、それに応じてdoesNotRecognizeSelector:メソッドがスローされ、これが冒頭で説明したクラッシュの根本的な原因となります。

注意深い読者の中には、クラッシュのスタックトレースの3行目にある**__ASPECTS_ARE_BEING_CALLED__**という行を見て、クラッシュがAspectsに関係している可能性があることに気付いた方もいるかもしれません。ここですべての試みをリストアップした理由は、ソースコードのないサードパートのフレームワークから、静的解析と動的解析によって問題を特定する方法を学んでいただきたいからです。この記事で紹介したトリックや技術があなたのお役に立てることを願っています。

#### Solution

このクラッシュを修正する方法は2つあります。1つは、**Aspects**のメソッドをフックすることで、例えばMethod Swizzlingのような侵襲性の低い方法で、**TencentOpenAPI**のメッセージフォワーディング処理中のセッター作成が中断されないようにします。もう一つは、`forwardInvocation:`を我々の実装に置き換えることで、`aliasSelector`と`originalSelector`の両方がメッセージ転送に応答できない場合、メッセージ転送のパスを元のパスに戻すことができます。以下のコードを参照してください。

```objective-c
     if (!respondsToAlias) {
          invocation.selector = originalSelector;
          SEL originalForwardInvocationSEL = NSSelectorFromString(AspectsForwardInvocationSelectorName);
         ((void( *)(id, SEL, NSInvocation *))objc_msgSend)(self, originalForwardInvocationSEL, invocation);
      }
```

実は、**Aspects** は **JSPatch** とコンフリクトしています。この2つのSDKの実装も似ているので、これらを一緒に使うと、`doesNotRecognizeSelector:`も起こります。微信读书的文章](http://wereadteam.github.io/2016/06/30/Aspects/)をご参照ください。

#### A perfect crush between Aspects and TencentOpenAPI

このクラッシュの根本的な原因は、**Aspects**と**TencentOpenAPI**のフレームワーク間の衝突である。UIViewController`クラスのライフサイクルメソッドはAspectsによってフックされており、`forwardInvocation`メソッドはAspectsの実装に置き換えられている。また、`TCWebViewController`のスーパークラスが`UIViewController`クラスであることから、`QQforwardInvocation`メソッドはAspectsの実装に置き換えられる。その結果、`TCWebViewController`クラスの`QQforwardInvocation`メソッドもAspectsにフックされることになる。その結果、メッセージ転送処理が失敗し、ゲッターやセッターの作成にも失敗してしまうのです。

このケースは、サードパーティ製フレームワークの使い方を学ぶだけでなく、そのメカニズムを調べる必要があることを教えてくれます。そうすれば、作業中に遭遇した問題を簡単に解決することができます。

## Summary

この記事ではさまざまなヒントを紹介していますが、デバッグ時の考え方もぜひマスターしてください。技術は簡単に習得できますが、問題解決のための考え方は簡単には形成できません。時間をかけて練習する必要があります。デバッグのテクニックに加えて、問題分析のセンスも必要です。そうすれば、問題はあなたにとって便利なものになるでしょう。

## Reference Material

* 《Code Complete》
* 《64 Bit Intel Assembly Language Programming for Linux》
* 《Debugging: The 9 Indispensable Rules for Finding Even the Most Elusive Software and Hardware Problems》
* 《Advanced Apple Debugging & Reverse Engineering》
* 《Computer Systems: A Programmer's Perspective》
* 《Debug It!: Find, Repair, and Prevent Bugs in Your Code》
* 《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs》
* [Principles of Objective-C Message Sending and Forwarding Mechanisms](http://yulingtianxia.com/blog/2016/06/15/Objective-C-Message-Sending-and-Forwarding/)

## Author

* [Alex Ao](https://github.com/aozhimin)
* [NewDu](https://github.com/NewDu)

## Acknowledgements

Special thanks to below readers, I really appreciate your support and valuable suggestions. 

* [ZenonHuang](https://github.com/ZenonHuang)
