# 動作環境 <!-- Requirements -->

<!--
Hard dependencies:
-->
必須の環境:

<!--
 * One of glibc, musl libc, uclib or bionic as your C library
 * Linux kernel >= 2.6.32
 -->
 * C ライブラリとして glibc、musl libc、uclib、bionic のいずれか
 * Linux カーネル 2.6.32 以上

<!--
Extra dependencies for lxc-attach:
-->
lxc-attach の動作に必要な環境:

<!--
 * Linux kernel >= 3.8
 -->
 * Linux カーネル 3.8 以上

<!--
Extra dependencies for unprivileged containers:
-->
非特権のコンテナが動作するのに必要な環境:

<!--
 * libpam-cgfs, cgmanager or another CGroup manager configuring your system for unprivileged CGroups operation
 * A recent version of shadow including newuidmap and newgidmap
 * Linux kernel >= 3.12
 -->
 * libpam-cgfs, cgmanager もしくは非特権の cgroup の操作が可能な他の cgroup マネージャ
 * newuidmap、newgidmap を含む最新バージョンの shadow
 * Linux カーネル 3.12 以上

<!--
Recommended libraries:
-->
推奨ライブラリ:

<!--
 * libcap (to allow for capability drops)
 * libapparmor (to set a different apparmor profile for the container)
 * libselinux (to set a different seccomp context for the container)
 * libseccomp (to set a seccomp policy for the container)
 * libgnutls (for various checksumming)
 * liblua (for the LUA binding)
 * python3-dev (for the python3 binding)
 -->
 * libcap (ケーパビリティを落とすために必要)
 * libapparmor (コンテナに対して独自の apparmor プロファイルを設定するために必要)
 * libselinux (コンテナに対して独自の SELinux コンテキストを設定するために必要)
 * libseccomp (コンテナに対して seccomp ポリシーを設定するために必要)
 * libgnutls (色々なチェックサム確認に必要)
 * liblua (lua バインディングに必要)
 * python3-dev (python3 バインディングに必要)

# インストール<!-- Installation -->

<!--
In most cases, you'll find recent versions of LXC available for your Linux distribution.
Either directly in the distribution's package repository or through some backport channel.
-->
通常はあなたがお使いのディストリビューションが、ディストリビューションのパッケージリポジトリもしくはバックポート用のチャンネル経由で、最新版の LXC を提供しているでしょう。

<!--
For your first LXC experience, we recommend you use a recent supported release,
such as a recent bugfix release of LXC 3.0.
-->
最初に LXC を使う場合は、LXC 3.0 の最新のバグフィックスのなされたバージョンのような、最新のサポート版リリースをお使いになることを推奨します。

<!--
If using Ubuntu, we recommend you use Ubuntu 18.04 LTS as your container host.
LXC bugfix releases are available directly in the distribution package repository
shortly after release and those offer a clean (unpatched) upstream experience.
-->
Ubuntu を使っている場合、コンテナホストとして Ubuntu 18.04 LTS を使うことを推奨します。
LXC のバグフィックスリリースは、リリース後すぐに直接ディストリビューションのパッケージリポジトリ経由で利用可能で、パッチの当たっていないクリーンな最新版を提供します。

<!--
Ubuntu is also one of the few (if not only) Linux distributions to come by default
with everything that's needed for safe, unprivileged LXC containers.
-->
Ubuntu は、安全な非特権の LXC コンテナのために必要な全てをデフォルトで揃えている Linux ディストリビューションのいくつかのうちの 1 つです (Ubuntu 以外にもそのようなディストリビューションは存在します)。

<!--
On such an Ubuntu system, installing LXC is as simple as:
-->
Ubuntu では、LXC をインストールするのは次のように簡単です:

    sudo apt-get install lxc or sudo snap install lxc

<!--
Your system will then have all the LXC commands available, all its templates
as well as the python3 binding should you want to script LXC.
-->
あなたのシステム上には、利用可能な LXC コマンドの全て、テンプレートの全て、LXC 処理のスクリプトに必要な python3 バインディングがインストールされるでしょう。


# 非特権コンテナの作成 <!-- Creating unprivileged containers as a user -->

<!--
Unprivileged containers are the safest containers.
Those use a map of uid and gid to allocate a range of uids and gids to a container.
That means that uid 0 (root) in the container is actually something like uid 100000
outside the container. So should something go very wrong and an attacker manages
to escape the container, they'll find themselves with about as many rights as a nobody user.
-->
非特権コンテナは最も安全なコンテナです。
非特権コンテナでは、コンテナで使う範囲の uid と gid を割り当てるために、uid と gid のマッピングを使います。
これはコンテナ内の uid 0 (root) が、コンテナの外では実際は uid が 100000 を持つというようになります。
それゆえ、万が一間違って問題のある操作を行ったり、攻撃者がコンテナを抜けだそうとしても、nobody ユーザと同じ程度の権限しか自身にないことがわかるでしょう。

<!--
Unfortunately this also means that the following common operations aren't allowed:
-->
残念ながら、このことは同時に以下のような操作が非特権コンテナでは許可されないことを意味します:

<!--
  * mounting most of filesystems
  * creating device nodes
  * any operation against a uid/gid outside of the mapped set
  -->
  * ほとんどのファイルシステムのマウント
  * デバイスノードの作成
  * マッピングが存在していない uid/gid に対する操作

<!--
Because of that, most distribution templates simply won't work with those.
Instead you should the "download" template which will provide you with pre-built images
of the distributions that are known to work in such an environment.
-->
このため、ほとんどのディストリビューションのコンテナテンプレートは動作しないでしょう。
代わりに、このような非特権の環境でも動くことを確認した、あらかじめビルド済みのディストリビューションのイメージを提供する "download" テンプレートを使う必要があります。

<!--
Now, everything below assumes a recent Ubuntu system or another Linux distribution which offers
a similar experience (recent kernel, recent version of shadow, cgmanager and default uid/gid allocation).
-->
現在、最新の Ubuntu か最新の Ubuntu と同じような環境 (最新のカーネル、最新バージョンの shadow、cgmanager、デフォルトの uid/gid 割り当て) の他の Linux ディストリビューションで、同様に以下のような操作ができるはずです。

<!--
First of all, you need to make sure your user has a uid and gid map defined in /etc/subuid and /etc/subgid.
On Ubuntu systems, a default allocation of 65536 uids and gids is given to every new user on the system,
so you should already have one. If not, you'll have to use usermod to give yourself one.
-->
まず第一に、お使いの (非特権コンテナを使おうとする) ユーザが /etc/subuid と /etc/subgid で定義された uid/gid のマッピングを持っている必要があります。
Ubuntu では、デフォルトで 65536 個の uid と gid の割り当てが、システム上で全ての新規ユーザに与えられますので、Ubuntu をお使いの場合はすでにそのマッピングを持っているはずです。
もしマッピングがない場合は、usermod コマンドを使って割り当てる必要があります。

<!--
Next up is /etc/lxc/lxc-usernet which is used to set network devices quota for unprivileged users.
By default, your user isn't allowed to create any network device on the host, to change that, add:
-->
次に、非特権ユーザに与えるネットワークデバイスの範囲を設定するために使う /etc/lxc/lxc-usernet を設定します。
デフォルトでは、ホスト上で全くネットワークデバイスを割り当てできないことになっていますので、このファイルに以下のような設定を追加します:

    your-username veth lxcbr0 10

<!--
This means that "your-username" is allowed to create up to 10 veth devices connected to the lxcbr0 bridge.
-->
これは、"your-username" にブリッジ lxcbr0 に接続する 10 個の veth デバイスの作成を許可するという意味です。


<!--
With that done, the last step is to create an LXC configuration file.
-->
これが済むと、最後のステップは LXC の設定ファイルの作成です。

<!--
 * Create the ~/.config/lxc directory if it doesn't exist.
 * Copy /etc/lxc/default.conf to ~/.config/lxc/default.conf
 * Append the following two lines to it:
    * lxc.idmap = u 0 100000 65536
    * lxc.idmap = g 0 100000 65536
 -->
 * ~/.config/lxc ディレクトリがない場合は作成します。
 * /etc/lxc/default.conf を ~/.config/lxc/default.conf にコピーします。
 * 以下の 2 行をコピーしたファイルに追加します。
    * lxc.idmap = u 0 100000 65536
    * lxc.idmap = g 0 100000 65536

<!--
Those values should match those found in /etc/subuid and /etc/subgid, the values above are those expected
for the first user on a standard Ubuntu system.
-->
ここで設定した値は /etc/subuid と /etc/subgid にある値と一致している必要があり、標準の Ubuntu システムの初期ユーザのために存在が必要です。

<!--
Just before you create your first container, you probably should logout and login again,
or even reboot your machine to make sure that your user is placed in the right cgroups.
(This is only required if cgmanager wasn't installed on your machine prior to you installing LXC.)
-->
最初のコンテナを作成するまえに、おそらく一度ログアウトしてログインするか、マシンのリブートが必要になるでしょう。これはお使いのユーザが使う cgroup が正しく作成されていることを確実にするためです。
(これは LXC をインストールする前にあなたのマシンに cgmanager がインストールされていなかった場合のみ必要なことです。)

<!--
And now, create your first container with:
-->
さて、以下のようにコンテナを作ってみましょう:

    lxc-create -t download -n my-container

<!--
The download template will show you a list of distributions, versions and architectures to choose from.
A good example would be "ubuntu", "bionic" (18.04 LTS) and "i386".
-->
ダウンロードテンプレートが選択できるディストリビューション、バージョン、アーキテクチャのリストを表示します。
例として一番良いのは "ubuntu"、"bionic" (18.04 LTS)、"i386" でしょう。

<!--
A few seconds later your container will be created and you can start it with:
-->
しばらくすると、コンテナが作成されますので、以下のように実行します:

    lxc-start -n my-container -d

<!--
You can then confirm its status with either of:
-->
実行したコンテナのステータスは以下のどちらかで確認できます:

    lxc-info -n my-container
    lxc-ls -f

<!--
And get a shell inside it with:
-->
コンテナ内でシェルを実行するには以下のようにします:

    lxc-attach -n my-container

<!--
Stopping it can be done with:
-->
コンテナの停止は以下のように行います:

    lxc-stop -n my-container

<!--
And finally removing it with:
-->
最後にコンテナを消去するには以下のようにします:

    lxc-destroy -n my-container

# root で非特権コンテナを作成する <!-- Creating unprivileged containers as root -->

<!--
To run a system-wide unprivileged container (that is, an unprivileged container started by root)
you'll need to follow only a subset of the steps above.
-->
システム全体で非特権コンテナを実行するには (これは root が非特権コンテナを実行するということです)、以下のような前述のステップの一部が必要なだけです。

<!--
Specifically, you need to manually allocate a uid and gid range to root in /etc/subuid and /etc/subgid.
And then set that range in /etc/lxc/default.conf using lxc.idmap entries similar to those above.
-->
具体的に言うと、root に対して割り当てる uid と gid の範囲を /etc/subuid と /etc/subgid に割り当てる必要があります。
そして、その範囲を先と同様に /etc/lxc/default.conf に lxc.idmap を使って設定します。

<!--
And that's it. Root doesn't need network devices quota and uses the
global configuration file so the other steps don't apply.
-->
以上です。root はネットワークデバイスの範囲を設定する必要はありません。グローバルの設定ファイルの設定を使いますので、このステップは不要です。

<!--
Any container you create as root from that point on will be running unprivileged.
-->
このようにして作成したコンテナは非特権コンテナとして動作するでしょう。

# 特権コンテナ <!-- Creating privileged containers -->

<!--
Privileged containers are containers created by root and running as root.
-->
特権コンテナは root が作成し、root が実行します。

<!--
Depending on the Linux distribution, they may be protected by some capability dropping, apparmor profiles,
selinux context or seccomp policies but ultimately, the processes still run as root and so you should never
give access to root inside a privileged container to an untrusted party.
-->
ディストリビューションによっては、特権コンテナはケーパビリティをいくつか落としたり、apparmor プロファイルや、SELinux コンテキスト、seccomp ポリシーでプロテクトされているかもしれません。しかし、最終的にはプロセスは root 権限で実行されますので、信頼できないユーザに特権コンテナ内の root 権限を与えるべきではありません。



<!--
If you still have to create privileged containers, it's quite simple. Simply don't do any of the configuration
described above and LXC will create privileged containers.
-->
特権コンテナを作成する必要がある場合は、非常に簡単です。単純に前述のようなステップを踏むことなく、特権コンテナが作成されます。

<!--
So:
-->

    sudo lxc-create -t download -n privileged-container

<!--
Will create a new "privileged-container" privileged container on your system using an image from the download template.
-->

以上のコマンドで、ダウンロードテンプレートからのイメージを使って、システム上に新しい "privileged-container" という名前の特権コンテナが作成されるでしょう。
