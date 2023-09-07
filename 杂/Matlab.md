调用python文件：

命令行中输入

```matlab
command = 'python test.py 1 2';
status = system(command);
```



用BP网络训练。

```matlab
%% 清空环境变量
clc
clear all
%% 训练数据预测数据提取及归一化
%% 训练集/测试集产生
%一列为一个样本
NIR= xlsread('wa.xlsx','A1:E46');%训练数据集的输入
octane=xlsread('wa.xlsx','F1:F46');%训练数据集的输出
% 随机产生训练集和测试集
temp = randperm(size(NIR,1));
% 训练集——40个样本
P_train = NIR(temp(1:40),:)';
T_train = octane(temp(1:40),:)';
% 测试集——6个样本
P_test = NIR(temp(41:end),:)';
T_test = octane(temp(41:end),:)';
N = size(P_test,2);

%[inputn,minp,maxp,outputn,mint,maxt]=premnmx(P_train,T_train);%归一化处理
[inputn, ps_input] = mapminmax(P_train,0,1);%归一化
input_n = mapminmax('apply', P_test, ps_input);
[target, ps_target] = mapminmax(T_train,0,1);%归一化
%训练样本输入输出数据归一化

%[inputn,minp,maxp,outputn,mint,maxt]=premnmx(input_train,output_train);%归一化处理

%disp(outputn)%看一下结果

%% BP网络训练
% %初始化网络结构
%确定隐含层节点个数
%采用经验公式hiddennum=sqrt(m+n)+a，m为输入层节点个数，n为输出层节点个数，a一般取为1-10之间的整数(3-12)
%net=newlin(inputn,target);
net=newff(minmax(P_train),[5,6,1],{'tansig','tansig','purelin'},'trainlm');%创建网络，表示有输入层5层，隐层5,8层，输出层1层
%net=newff(inputn,outputn,[5,1],{'logsig','purelin'},'trainlm','msereg');
%net=newff(inputn,outputn,[10,5],{'logsig','purelin'},'sse','trainlm');%创建网络，表示有输入层5层，两个隐藏层分别有10、5个神经元，输出层1层
% 变学习率梯度下降算法  
%net.trainFcn='traingda';
net.trainParam.epochs=2000;%设置最大收敛次数，把500改为了100
net.trainParam.lr=5;%设置学习速率
net.trainParam.goal=0.00001;%设置收敛误差
%net.trainParam.min_grad=1e-6;%设置最小性能梯度，一般取1e-6
%net.trainParam.min_fail=10;%设置最大确认失败次数

%网络训练
net=train(net,inputn,target);

%% BP网络预测
%预测数据归一化
%input_n=tramnmx(input_test,minp,maxp);

%网络预测输出
an=sim(net,input_n);

%网络输出反归一化
BPoutput = mapminmax('reverse', an, ps_target); %反归一化
%BPoutput=postmnmx(an,mint,maxt);
%% 结果分析
figure(1)%图一
plot(BPoutput,':o')
hold on
plot(T_test,'-*');
legend('预测输出','期望输出')
title('BP网络预测输出','fontsize',12)
ylabel('函数输出','fontsize',12)
xlabel('样本','fontsize',12)

%相对误差
error1=(abs(BPoutput-T_test))./T_test;
%标准

figure(2)
plot(error1,'-*')
title('BP网络预测相对误差','fontsize',12)
ylabel('相对误差','fontsize',12)
xlabel('样本','fontsize',12)



save my_BP net;%保存训练好的BP神经网络net
```

