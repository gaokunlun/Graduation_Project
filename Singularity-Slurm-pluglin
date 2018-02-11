##修复新版本Singularity对slurm的支持的问题

```shell
$ git clone https://github.com/singularityware/singularity.git下载github上最新的Singularity源码
$ version=<version>
$ git clone https://github.com/singularityware/singularity.git
$ cd singularity
$ git checkout tags/${version} -b ${version}
$ ./autogen.sh
```
同时下载Singularity-2.3.1源码包:
```shell
$ wget https://github.com/singularityware/singularity/releases/download/2.3.1/singularity-2.3.1.tar.gz
$ tar -xvzf singularity-2.3.1.tar.gz
```
将singularity中的configure文件替换为singularity-2.3.1中的configure文件，其中包含了with-slurm的支持：
```python
if test "${with_slurm+set}" = set; then :
  withval=$with_slurm;
fi

if test "x$with_slurm" == "xyes"; then :

        { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5
$as_echo "yes" >&6; }
        ac_fn_c_check_header_mongrel "$LINENO" "slurm/spank.h" "ac_cv_header_slurm_spank_h" "$ac_includes_default"
if test "x$ac_cv_header_slurm_spank_h" = xyes; then :
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5
$as_echo "yes" >&6; }
                with_slurm="yes"

else

                as_fn_error $? "SLURM support requested, but slurm/spank.h header not found." "$LINENO" 5

fi



else

        { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5
$as_echo "no" >&6; }
        with_slurm="no"

fi
 if test "$with_slurm" = "yes"; then
  WITH_SLURM_TRUE=
  WITH_SLURM_FALSE='#'
else
  WITH_SLURM_TRUE='#'
  WITH_SLURM_FALSE=
fi
```
新版本的Singularity源码src/slurm/中缺乏Makefile.in文件，亦将singularity-2.3.1/src/slurm/Makefile.in拷贝过去，然后回到singularity目录进行编译安装
```shell
$ cp singularity-2.3.1/src/slurm/Makefile.in singularity/src/slurm
$ cd singularity/
$ ./configure --prefix=/usr/local --with-slurm
$ make
$ sudo make install
```
编译过程中需要安装一些依赖包：
```shell
$ yum install -y gcc automake autoconf texinfo
$ wget http://ftp.gnu.org/gnu/automake/automake-1.14.1.tar.gz
$ tar -xvzf automake-1.14-1.tar.gz
$ cd automake-1.14.1
$ ./configure
$ make
$ sudo make install
```
Singularity安装完成后，在slurm配置文件目录/etc/slurm/下面的plugstack.conf配置文件中，添加：

required /usr/local/lib/slurm/singularity.so

此时，即可在slurm任务中使用singularity了
