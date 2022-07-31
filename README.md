# <center>SM4 optimal 实验报告</center>

>**课程名称     <u>创新创业实践课程</u>  **       
>
>**学生姓名   <u>李路岩</u>      学号  <u>202022180198</u>**     
>
>**学院   <u>网络空间安全</u>学院    专业  <u>信息安全</u>**   

[TOC]

>****

## <center>实验思路</center>

>**本次实验基于学期内系统原理课程的SM4实验**
>
>**1.多线程：**
>
><a href="https://img.gejiba.com/image/EyHRnU"><img src="https://img.gejiba.com/images/aa7ddcd646bd34a8af25831c4f0a2808.png" alt="aa7ddcd646bd34a8af25831c4f0a2808.png" border="0"></a>
>
>**2.SIMD优化：**
>
>​		在本次实验中，我们尝试使用 SIMD 来优化程序一达到减少运行时间的目的，但结果发现，通过使用 SIMD 相关指令集来优化并不能达到使时间减少的目的，反而使时间开销更大了。
>​		分析原因，我们推测:m128i 存的数据长度与 uint32 存的是相同的，最后取 m128 类型的时候还需要转换成一个数来进行运算；整个过程中，读取、再计算、再存、再转换，时间开销很大，不如直接计算。
>
>​		所以使用 SIMD 指令集对算法加速并无太大影响。  
>
>**3.循环展开**：
>
><a href="https://img.gejiba.com/image/EyHmv3"><img src="https://img.gejiba.com/images/3f89869727761ffce59ec9c6d4968ce8.png" alt="3f89869727761ffce59ec9c6d4968ce8.png" border="0"></a>
>
>**4.查表优化：**
>
><a href="https://img.gejiba.com/image/EyXBRw"><img src="https://img.gejiba.com/images/f693a1daffe377c32bb030dc725144d1.png" alt="f693a1daffe377c32bb030dc725144d1.png" border="0"></a>

## <center>关键代码</center>

```c++
//多线程 ：
for (int k = 0; k < 11; k++) {
        QueryPerformanceFrequency(&f);
        dqFreq = (double)f.QuadPart;
        QueryPerformanceCounter(&time_start);	//计时开始
        for (int i = 0; i < 8; i = i + 1)
        {
            t[i] = thread(solve, i * maxnum / 8, i);

            /*t[i+1] = thread(solve, (i+1) * maxnum / 8, i+1);*/
        }
        for (int i = 0; i < 8; i = i + 1)
        {
            t[i].join();
            /* t[i+1].join();
             t[i + 2].join();
             t[i+3].join();*/
        }
//循环展开：
    for (int a = i * maxnum / 8; a < (i + 1) * maxnum / 8; a++)
    {
        for (int i = 0; i < 4; i=i+2)
        {
            X[a][i] = plaintext[a][i];
            X[a][i+1] = plaintext[a][i+1];
        }
        for (int i = 0; i < 32; i=i+4)
        {
            X[a][i + 4] = X[a][i] ^ T(X[a][i + 1] ^ X[a][i + 2] ^ X[a][i + 3] ^ rkey[i]);
            X[a][i + 5] = X[a][i+1] ^ T(X[a][i + 2] ^ X[a][i + 3] ^ X[a][i + 4] ^ rkey[i+1]);
            X[a][i + 6] = X[a][i + 2] ^ T(X[a][i + +3] ^ X[a][i + 4] ^ X[a][i + 5] ^ rkey[i + 2]);
            X[a][i + 7] = X[a][i+3] ^ T(X[a][i + 4] ^ X[a][i + 5] ^ X[a][i + 6] ^ rkey[i+3]);
        }
        for (int i = 0; i < 4; i=i+2)
        {
            secrettext[a][i] = X[a][35 - i];
            secrettext[a][i+1] = X[a][34 - i];
        }
    }
//查表优化：
    uint32_t T1(uint32_t x)
{
    uint8_t s[4];
    uint32_t res = 0;
    for (int i = 0; i < 4; i=i+2)
    {
    
        s[i] = x >> (24 - i * 8);
        s[i] = Sbox[s[i] >> 4][s[i] & 0x0f];
        res |= s[i] << (24 - i * 8);

        s[i+1] = x >> (24 - (i+1) * 8);
        s[i+1] = Sbox[s[i+1] >> 4][s[i+1] & 0x0f];
        res |= s[i+1] << (24 - (i+1) * 8);
       /* s[i + 2] = x >> (24 - (i + 2) * 8);
        s[i + 2] = Sbox[s[i + 2] >> 4][s[i +2] & 0x0f];
        res |= s[i + 2] << (24 - (i + 2) * 8);

        s[i + 3] = x >> (24 - (i + 3) * 8);
        s[i + 3] = Sbox[s[i + 3] >> 4][s[i + 3] & 0x0f];
        res |= s[i + 3] << (24 - (i + 3) * 8);*/
        
    }
    return res ^ (((res << 13) | (res >> 19)) & num) ^ (((res << 23) | (res >> 9)) & num);
}
```





## <center>实验结果</center>

<a href="https://img.gejiba.com/image/EyXnUC"><img src="https://img.gejiba.com/images/e403e1a72fff5357cb2a2e0d628ded08.png" alt="e403e1a72fff5357cb2a2e0d628ded08.png" border="0"></a>
