## EIGEN理解与相关实验及BLAS+LAPACK对比
**1. 认识EIGEN**
>EIGEN是一个C++线性运算模板库，可以用来完成矩阵、向量、数值解等相关运算。

**2. 如何安装EIGEN**
>使用EIGEN不需要安装其他依赖，直接下载源码包http://bitbucket.org/eigen/eigen/get/3.3.3.tar.gz并解压，将目录中的Eigen子目录拷>贝到/usr/local/include/，项目中直接包含其中的头文件即可使用，免去使用Cmake进行编译的烦恼

**3. EIGEN特性**
- 通用性（versatile）：
  - 支持所有的矩阵量级，从小矩阵到任意大的稠密矩阵甚至稀疏矩阵
  - 支持所有的标准数据类型，包括std::complex, integers，扩展性强
  - 支持多种矩阵分解方法和几何特征
  - 在其不支持的模块中提供了许多特殊的特性，比如非线性优化、矩阵方程、多项式解等
- 快速（fast）：
  - 删除不必要的中间结果以及适当的惰性验证(lazy evaluation)
  - 利用SSE 2/3/4, AVX, FMA, AVX512, ARM NEON (32-bit and 64-bit), PowerPC AltiVec/VSX (32-bit and   64-bit)等指令集对非向量化的代码进行显示向量化
  - 对固定大小的矩阵的优化
  - 对于大矩阵，着重于cache的友好性
- 可靠性：
  - 根据每种算法的试用情形进行可靠性的trade-offs，并加以合理选择
  - EIGEN通过了所有自定义测试集（超过500个），以及标准的BLAS测试集，和部分LAPACK测试集
- 优雅性：
  - 对C++模板的表示，使其API接口既简明又具有很强的表达性，符合传统C++程序员的习惯
  - 在EIGEN上实现算法，就像写伪代码一样便捷
- 很好的编译支持：
  - 在大量的测试集上运行，保证了在任何情形下的编译的可靠性

**4. [官网](http://eigen.tuxfamily.org/dox/GettingStarted.html)一个简单的示例**

计算m\*v，其中m为矩阵，v为向量
```c++
//test.cpp
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;
using namespace std;
int main()
{
MatrixXd m = MatrixXd::Random(3,3);
m = (m + MatrixXd::Constant(3,3,1.2)) * 50;
cout << "m =" << endl << m << endl;
VectorXd v(3);
v << 1, 2, 3;
cout << "m * v =" << endl << m * v << endl;
}
```
编译程序：g++ test.cpp –o test得到test执行文件，运行./test：
```sh
m =
94.0188  89.844 43.5223
49.4383 101.165  86.823
88.3099 29.7551 37.7775
m*v =
404.274
512.237
261.153
```

**5. 求解线性方程组A\*x=b**

>求解线性方程组需要用到矩阵分解的方法，矩阵分解是将矩阵分解成数个矩阵的乘积，可分为三角分解、满秩分解、QR分解、Jordan分解、SVD奇异值分解等；常见的有三种：三角分解(Triangular Factorization)、QR分解法(QR Factorization)、奇异值分解(Singular Value Deposition)

- **LU三角分解**：
    >三角分解法是将原正方 (square) 矩阵分解成一个上三角形矩阵　或是排列(permuted) 的上三角形矩阵和一个 下三角形矩阵，这样的分解法又称为LU分解法。它的用途主要在简化一个大矩阵的行列式值的计算过程，求 反矩阵，和求解联立方程组。不过要注意这种分解法所得到的上下三角形矩阵并非唯一，还可找到数个不同 的一对上下三角形矩阵，此两三角形矩阵相乘也会得到原矩阵。

- **QR分解**：
    >QR分解法是将矩阵分解成一个正规正交矩阵与上三角形矩阵,所以称为QR分解法,与此正规正交矩阵的通用符号Q有关。

- **奇异值分解(SVD)**：
    >奇异值分解 (singular value decomposition,SVD) 是另一种正交矩阵分解法；SVD是最可靠的分解法，但是它比QR 分解法要花上近十倍的计算时间。[U,S,V]=svd(A)，其中U和V分别代表两个正交矩阵，而S代表一对角矩阵。 和QR分解法相同， 原矩阵A不必为正方矩阵。使用SVD分解法的用途是解最小平方误差法和数据压缩。

- **LLT分解**：
    >顾名思义，LLT分解的含义为A=LL^T，cholesky分解是把对称正定矩阵表示成一个下三角矩阵L和其转置乘积的分解。它要求矩阵的所有特征值必须大于零，故分解的下三角的对角元也必须大于零(LU三角分解的变形)。

- **LDLT分解**：
    >若A为一个对称矩阵且其任一k阶主子阵均不为零，则A有如下唯一的分解形式：
    A = LDL^T
    其中L为一个下三角单位矩阵，D为一个对角矩阵，L^T为L的转置
    LDLT是cholesky LLT分解法的改进，因为cholesky分解法虽然不需要选主元，但其运算过程中涉及到开方问题，而LDLT分解法则避免了这一问题，可用于求解线性方程组。

EIGEN中对应的分解方法为：
| Decomposition | Method | Requirements on the matrix | Speed(small to medium) | Speed(large) | Accuracy |
| ------------- | ------ | -------------------------- | ---------------------- | ------------ | -------- |
| PartialPivLU	| partialPivLu() | Invertible |	++ | ++ | + |
| FullPivLU | fullPivLu() | None | - | - - | +++ |
| HouseholderQR	| householderQr() | None | ++ | ++ | + |
| ColPivHouseholderQR | colPivHouseholderQr() | None | ++ | - | +++ |
| FullPivHouseholderQR | fullPivHouseholderQr() | None | - | - - | +++ |
| LLT | llt() | Positive definite | +++ | +++ | + |
| LDLT | ldlt() | Positive or negative semidefinite | +++ | + | ++ |
| JacobiSVD | jacobiSvd() | None | - - | - - - | +++ |


 **第一个例子**：
```c++
/*decomposition1.cpp*/
#include <iostream>
#include <Eigen/Dense>

using namespace std;
using namespace Eigen;

int main(){
     Matrix3f A;
     A << 1,2,3, 4,5,6, 7,8,10;
     cout << "The matrix A =  " << endl << A << endl;
     Vector3f b;
     b << 3, 3, 4;
     cout << "The vector b = " << endl << b <<endl;
     VectorXf x = A.llt().solve(b);
     cout << "The llt solustion of A*x = b is : " << endl << x <<endl;
     double llt_err = (A*x-b).norm() / b.norm();
     cout << "The relative error of llt solution is : " << llt_err << endl;

     VectorXf x1 = A.ldlt().solve(b);
     cout << "The ldlt solustion of A*x = b is : " << endl << x1 <<endl;
     double ldlt_err = (A*x1-b).norm() / b.norm();
     cout << "The relative error of ldlt solution is : " << ldlt_err << endl;

     VectorXf x2 = A.householderQr().solve(b);
     cout << "The householderQr solustion of A*x = b is : " << endl << x2 <<endl;
     double householderqr_err = (A*x2-b).norm() / b.norm();
     cout << "The relative error of householderQr solution is : " << householderqr_err << endl;

     VectorXf x3 = A.colPivHouseholderQr().solve(b);
     cout << "The colPivHouseholderQr solustion of A*x = b is : " << endl << x3 <<endl;
     double cphouseholderqr_err = (A*x3-b).norm() / b.norm();
     cout << "The relative error of colPivHouseholderQr solution is : " << cphouseholderqr_err << endl;
     return 0;
}
```
然后执行编译： g++ decomposition1.cpp -o decomposition1
```sh
./decomposition1
The matrix A =
 1  2  3
 4  5  6
 7  8 10
The vector b =
3
3
4
The llt solustion of A*x = b is :
 4.4556
-0.3184
 -0.026
The relative error of llt solution is : 4.74642
The ldlt solustion of A*x = b is :
-0.206897
  0.37931
 0.241379
The relative error of ldlt solution is : 0.307059
The householderQr solustion of A*x = b is :
      -2
0.999998
       1
The relative error of householderQr solution is : 4.08885e-07
The colPivHouseholderQr solustion of A*x = b is :
      -2
0.999997
       1
The relative error of colPivHouseholderQr solution is : 2.45331e-07
```
我们可以发现对于一般的矩阵A，householderQr()和colPivHouseholderQr()都可以取得比较好的结果，而A不是(半)正定\负定阵的情况下，LLT和LLDT方法虽然也能结果，但是准确率比较差。

**第二个例子**：

此时矩阵A不是方阵，且A\*x = b存在解：
```c++
/*decompostion2.cpp*/
#include <iostream>
#include <Eigen/Dense>

using namespace std;
using namespace Eigen;

int main(){
    MatrixXf A(4,3);
    A << 1,2,3, 4,5,6, 3,1,2, 7,8,10;
    cout << "The matrix A =  " << endl << A << endl;
    Vector4f b(4);
    b << 1, 2, 3, 5;
    
    VectorXf x = A.householderQr().solve(b);
    cout << "The householderqr solution of A*x = b is : " << endl << x <<endl;
    double h_err = (A*x-b).norm() / b.norm();
    cout << "The householderqr error is : " << h_err << endl;

    VectorXf x1 = A.colPivHouseholderQr().solve(b);
    cout << "The colPivHouseholderqr solution of A*x = b is : " << endl << x1 <<endl;
    double cph_err = (A*x1-b).norm() / b.norm();
    cout << "The colPivHouseholderqr error is : " << cph_err << endl;

    VectorXf x3 = A.fullPivLu().solve(b);
    cout << "The fullPivLu solution of A*x = b is : " << endl << x3 <<endl;
    double flu_err = (A*x3-b).norm() / b.norm();
    cout << "The fullPivLu error is : " << flu_err << endl;
    return 0;
}
```
然后执行编译： g++ decomposition2.cpp -o decomposition2
```sh
./decomposition2
The matrix A =
 1  2  3
 4  5  6
 3  1  2
 7  8 10
The householderqr solution of A*x = b is :
0.825095
-1.37643
0.996199
The householderqr error is : 0.0789914
The colPivHouseholderqr solution of A*x = b is :
0.825095
-1.37643
  0.9962
The colPivHouseholderqr error is : 0.0789914
The fullPivLu solution of A*x = b is :
0.846154
-1.07692
0.769231
The fullPivLu error is : 0.0985404
```
因此，我们发现householderQr()、colPivHouseholderQr()、fullPivLu()并不要求矩阵A本身具有可逆、正定等特殊的性质，就可以达到很好的准确率。且householderQr()、colPivHouseholderQr()更优一些。

**第三个例子**：

此时，定义矩阵A为一个对称正定阵： 
```c++
/*decomposition3.cpp*/
#include <iostream>
#include <Eigen/Dense>

using namespace std;
using namespace Eigen;

int main(){
    Matrix3f A;
    A << 2, 1, 1, \
        1, 2, -1, \
        1, -1, 3;
    cout << "The matrix A =  " << endl << A << endl;
    Vector3f b;
    b << 1, 2, 3;
    
    VectorXf x = A.llt().solve(b);
    cout << "The llt solustion of A*x = b is : " << endl << x <<endl;
    double llt_err = (A*x-b).norm() / b.norm();
    cout << "The relative error of llt solution is : " << llt_err << endl;

    VectorXf x1 = A.ldlt().solve(b);
    cout << "The ldlt solustion of A*x = b is : " << endl << x1 <<endl;
    double ldlt_err = (A*x1-b).norm() / b.norm();
    cout << "The relative error of ldlt solution is : " << ldlt_err << endl;

    VectorXf x2 = A.householderQr().solve(b);
    cout << "The householderqr solution of A*x = b is : " << endl << x2 <<endl;
    double h_err = (A*x2-b).norm() / b.norm();
    cout << "The householderqr error is : " << h_err << endl;
 
    VectorXf x3 = A.colPivHouseholderQr().solve(b);
    cout << "The colPivHouseholderqr solution of A*x = b is : " << endl << x3 <<endl;
    double cph_err = (A*x3-b).norm() / b.norm();
    cout << "The colPivHouseholderqr error is : " << cph_err << endl;
    return 0;
}
```
然后执行编译： g++ decomposition3.cpp -o decomposition3
```sh
./decomposition3
The matrix A =
 2  1  1
 1  2 -1
 1 -1  3
The llt solustion of A*x = b is :
-4
 5
 4
The relative error of llt solution is : 5.25449e-07
The ldlt solustion of A*x = b is :
-4
 5
 4
The relative error of ldlt solution is : 2.84965e-07
The householderqr solution of A*x = b is :
-4
 5
 4
The householderqr error is : 4.59492e-07
The colPivHouseholderqr solution of A*x = b is :
-4
 5
 4
The colPivHouseholderqr error is : 2.38419e-07
```
此时，对比发现LLT、LDLT可以给出很准确的解，且householderqr()、colPivHouseholderqr()依旧能有很不错的表现

**6. BLAS/APACK求解 A\*x = b**

- **编译blas：**
    ```sh
    wget http://www.netlib.org/blas/blas-3.7.0.tgz
    tar –xzvf blas-3.7.0.tgz
    cd BLAS-3.7.0/
    gfortran -c -O3 *.f  #编译生成.o执行文件
    ar rv libblas.a *.o  #链接所有的 .o文件，生成 .a 文件
    cp libblas.a /usr/local/lib  # 将库文件复制到系统库目录
    ```
    
- **编译cblas：**
    ```sh
    wget http://www.netlib.org/blas/blast-forum/cblas.tgz
    tar –xzvf cblas.tgz
    cd CBLAS/
    cp Makefile.LINUX Makefile.in
    cp ../BLAS-3.7.0/libblas.a testing/ #将上一步编译成功的libblas.a复制到testing/目录下
    make #编译所有目录
    cp lib/cblas_LINUX.a /usr/local/lib/libcblas.a # 将库文件复制到系统库目录下
    ```

- **编译lapack和lapacke**
    ```sh
    wget http://www.netlib.org/lapack/lapack-3.7.0.tgz
    tar –xvzf lapack-3.7.0.tgz
    mv lapack-3.7.0/ /usr/local/src/
    cd /usr/local/src/lapack-3.7.0/
    cp INSTALL/make.inc.gfortran make.inc
    ```
    对make.inc进行编辑
    ```sh
    BLASLIB      = /usr/local/lib/librefblas.a
    CBLASLIB     = /usr/local/lib/libcblas.a
    LAPACKLIB    = liblapack.a
    TMGLIB       = libtmglib.a
    LAPACKELIB   = liblapacke.a
    ```
    ```sh
    make #编译所有lapack文件
    
    cd LAPACKE #进入LAPACKE 文件夹，这个文件夹包含lapack的C语言接口文件
    make #编译所有lapacke文件
    cp include/*.h /usr/local/include
    cd ..
    cp *.a /usr/local/lib
    ```
    至此，BLAS及LAPACK安装完毕

- **编写C源程序，计算A\*=b**
    ```cpp
    #include <stdio.h>
    
    //lapacke headers
    #include "lapacke.h"
    #include "lapacke_config.h"
    #include "lapacke_utils.h"
    
    extern lapack_int LAPACKE_dgesv( int matrix_order, lapack_int n, lapack_int nrhs,
                              double* a, lapack_int lda, lapack_int* ipiv,
                              double* b, lapack_int ldb );
    
    int main()
    {
        int N = 4;
        double A[16] = {  1,  2,  3,  1,
                          4,  2,  0,  2,
                         -2,  0, -1,  2,
                          3,  4,  2, -3};
        double B[8] = {  6,  2,  1,  8,
                         1,  2,  3,  4};
        int ipiv[4];
        int n = N;
        int nrhs = 2;
        int lda = N;
        int ldb = N;
    
        int info = LAPACKE_dgesv(LAPACK_COL_MAJOR,n,nrhs,A,lda,ipiv,B,ldb);
        printf("info:%d\n",info);
        if(info==0)
        {
            int i = 0;
            int j = 0;
            for(j=0;j<nrhs;j++)
            {
                printf("x%d\n",j);
                for(i=0;i<N;i++)
                    printf("%.6g \t",B[i+j*N]);
                printf("\n");
            }
        }
    }
    ```

    运行结果为：
    info:0
    x0
    1 2 0 -1 
    x1
    1.375 0.375 0.375 -0.375
    表示的是
    A = 1 4 -2 3
    2 2 0 4
    3 0 -1 2
    1 2 2 -3
    B = 6 1
    2 2 
    1 3
    8 4
    x = 1 1.375
    2 0.375
    0 0.375
    -1 -0.375

**7. EIGEN VS BLAS/LAPACK**
官方的[slides](http://downloads.tuxfamily.org/eigen/eigen_plafrim_may_2011.pdf)中指出：
>• Pros:
– C++ friendly API
– Matrix manipulation
– Easy to use, install, distribute, etc.
– Custom scalar types
– Static allocation
– Temporary removal
– Auto vectorization, ARM NEON
– Higher perf for small objects
– etc.
• Cons:
– Covers only most common features of lapack
– Not fully multi-threaded yet

**8. 总结**
 通过以上实验， 我们也可以看出无论是C/C++ API接口的友好性、安装及编程的简便性，对于不同类型矩阵运算的灵活性等，EIGEN都比BLAS/LAPACK更胜一筹