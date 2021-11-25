- 用matlab实现序列

  - x(n) = 0.8^n^ , 0 ≤ n ≤ 15

    ```matlab
    n=[0:15];
    x=(0.8).^n;
    stem(n,x);
    ```

  - x(n) = e^(0.2+3j)n^ , 0 ≤ n ≤ 15

    ```matlab
    n = [0:15];
    x = exp((0.2+3j)*n);
    stem(n,x);
    ```

  - x(n) = 3cos(0.125$\pi$n+0.2$\pi$)+2sin(0.25$\pi$n+0.1$\pi$) , 0 ≤ n ≤ 15

    ```matlab
    n = [0:15];
    x = 3*cos(0.125*pi*n + 0.2*pi)+2*sin(0.25*pi*n + 0.1*pi);
    stem(n,x);
    ```

  - 将上式中的x(n)扩展为以10为周期的函数x~10~(n) = x(n+10)，绘制出四个周期

    ```matlab
    n = [0:9];
    x = 3*cos(0.125*pi*n + 0.2*pi)+2*sin(0.25*pi*n + 0.1*pi);
    xtilde=[x,x,x,x];
    n=[0:39];
    stem(n,xtilde);
    ```

- 已知一因果系统的差分方程为6y(n)+2y(n-2) = x(n)+3x(n-1)+3x(n-2)+x(n-3)，在该系统的输入端加一个矩形脉冲序列，其脉冲宽度与周期的比例（即占空比）为1：2，一个周期取16 个采样点，取三个周期，求该系统的输出

  - 实现矩形脉冲序列

    ```matlab
    n=[0:47];
    x=[ones(1,8),zeros(1,8)];
    x=[x,x,x];
    
    subplot(2,2,1);stem(n,x);axis([0 47 0 1.2]);
    xlabel('n');ylabel('x(n)');
    ```

  - 实现差分方程

    ```matlab
    a=[6,0,2];
    b=[1,3,3,1];
    y=filter(b,a,x);
    
    subplot(2,2,2);stem(n,y);axis([0 47 -0.5 1.5]);
    xlabel('n');ylabel('y(n)');title('函数方法');
    ```

  - 矩形脉冲序列的冲激响应

    ```matlab
    h=impz(b,a,n);
    
    subplot(2,2,3);stem(n,h);axis([0 47 -0.2 0.6]);
    xlabel('n');ylabel('h(n)');
    ```

  - 卷积得到输出

    ```matlab
    y=conv(x,h);
    n=[0:length(y)-1];
    
    subplot(2,2,4);stem(n,y);axis([0 47 -0.5 1.5]);
    xlabel('n');ylabel('y(n)');title('卷积方法');
    ```

- 已知两个序列为f~1~(n) = 0.8^n^ (0<n<20), f~2~(n) = u(n) (0<n<10)，求两序列的卷积和

  ```matlab
  n1 = [0:20];
  n2 = [0:10];
  f1 = 0.8.^n1;
  f2 = 1;
  f = conv(f1,f2);
  
  stem(f);title('f');xlabel('n');ylabel('f');
  ```

- 已知离散时间系统的系统函数为$H(z) = \frac{0.1-0.4z^{-1}+0.4z^{-2}-0.1z^{-3}}{1+0.3z^{-1}+0.55z^{-2}+0.2z^{-3}}$，求该系统在0到$\pi$频率范围内，归一化的绝对幅度频率响应、相位频率响应和零极点分布图

  - 零极点分布图

    ```matlab
    b = [0.1, -0.4, 0.4, -0.1];
    a = [1, 0.3, 0.55, 0.2];
    zplane(b,a)
    ```

  - 绝对频率响应

    ```matlab
    w=[0:1:100]*pi/100;
    H=freqz(b,a,w);
    magH=abs(H);
    
    subplot(3,1,1);plot(w/pi, magH);grid
    xlabel('frequency in pi units');ylabel('幅度');
    ```

  - 相对幅度频率响应

    ```matlab
    magJ=20*(log(magH)/log(10));
    
    subplot(3,1,2);plot(w/pi,magJ);grid
    xlabel('frequency in pi units');ylabel('幅度');

  - 相对相位频率响应

    ```matlab
    phaH=angle(H);
    
    subplot(3,1,3);plot(w/pi,phaH/pi);grid
    xlabel('frequency in pi units');ylabel('相位');

- 已知一阶离散系统的传递函数为H(z) = $\frac{z-q}{z-p}$，①：假设系统的零点在原点，极点分别取0.2，0.5，0.8，比较它们的幅频响应曲线②：假设系统的极点在原点，零点分别取0.2，0.5，0.8，比较它们的幅频响应曲线

  - 传递函数

    ```matlab
    % 第一问
    b=[1,0];
    % 第二问
    a=[1,0];
    
    w=[0:1:100]*pi/100;

  - 极点取0.2

    ```matlab
    a1=[1,-0.2];
    H=freqz(b,a1,w);
    magH=abs(H); 
    
    subplot(3,1,1);plot(w/pi,magH);grid
    xlabel('frequency in pi units');ylabel('幅度');
    ```

  - 极点取0.5

    ```matlab
    a2=[1,-0.5];
    I=freqz(b,a2,w);
    magI=abs(I); 
    
    subplot(3,1,2);plot(w/pi,magI);grid
    xlabel('frequency in pi units');ylabel('幅度');
    ```

  - 零点取0.2

    ```matlab
    b1=[1,-0.2];
    H=freqz(b1,a,w);magH=abs(H); 
    
    subplot(3,1,1);plot(w/pi,magH);grid
    xlabel('frequency in pi units');ylabel('幅度');

  - 零点取0.5

    ```matlab
    b2=[1,-0.5];
    I=freqz(b2,a,w);magI=abs(I); 
    
    subplot(3,1,2);plot(w/pi,magI);grid
    xlabel('frequency in pi units');ylabel('幅度');

  - 其余略

  > (不能正常运行)若x(n) = $sin(\frac{n\pi}{4})$是一个N=32的有限序列，计算它的DFT并画出图形。

  

