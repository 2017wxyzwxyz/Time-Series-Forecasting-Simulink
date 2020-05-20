# GRU�w���܂񂾃j���[�����l�b�g���[�N�ɂ�鐅�������������ڃ��f���̍\�z
# �͂��߂�


�����́A41.5�N���̐������Ґ������߂� MAT�t�@�C���uchickenpox_dataset.mat�v�ɂ���f�[�^�ɑ΂��āA�ċA�I�ȃj���[�����l�b�g���[�N�����̑w�̂P�� GRU (Gated Recurrent Unit) ��K�p���ė\�����f�����\�z���܂��B���A������ Deep Learning Toolbox �ɕt��������̂P�u[Time Series Forecasting Using Deep Learning](https://www.mathworks.com/help/deeplearning/examples/time-series-forecasting-using-deep-learning.html)�v�Ɋ�Â��Ă���܂�


# GRU �ɂ���


GRU �́A�ċA�I�ȃj���[�����l�b�g���[�N��\������w�̈���  LSTM (Long Short Term Memory) ���V���v���ȍ\���������Ȃ��� Elman �^�l�b�g���[�N�Ƃ͈Ⴂ�A���ݎ��� <img src="https://latex.codecogs.com/gif.latex?\inline&space;t"/> ����̉��߂ɔ����ďd�݂�ς��Ă���Ƃ��������������܂�


  


![image_0.png](README_images/image_0.png)


  
  


![image_1.png](README_images/image_1.png)


  
# 1: MAT-�t�@�C������f�[�^���C���|�[�g


�����̐������Ґ������߂��uchickenpox_dataset.mat�v����f�[�^���C���|�[�g���܂�



```matlab:Code
matobj = matfile('chickenpox_dataset.mat');
RawDataCell = matobj.chickenpoxTargets;
RawData = [RawDataCell{:}]';
T = 1:numel(RawData);
```

# 2: �������Ҕ��������������ŉ���


�����̐������Ґ����O���t�\�����܂�



```matlab:Code
figure
plot(RawData)
xlabel("Month")
ylabel("Cases")
title("Monthly Cases of Chickenpox")
```


![figure_0.png](README_images/figure_0.png)

# 3: �f�[�^�̐��K��
  


![image_2.png](README_images/image_2.png)


  
  


Statistics and Machine Learning Toolbox ���񋟂���uzscore�v�֐��𗘗p����ƕ��ς������ĕW���΍��Ŋ���Ƃ������K���̍�Ƃ��P�s�ōς܂��邱�Ƃ��ł��܂�



```matlab:Code
[nData, mu, sigma] = zscore(RawData);
```

# 4: �f�[�^�̕���


�O���̖�90%���w�K�p�̃f�[�^�Ƃ��āA�㔼�̖�10%�����ؗp�̃f�[�^�Ƃ��ĕ������܂�



```matlab:Code
NumTrain = floor(0.9*numel(nData));
TrainData = nData(1:NumTrain+1);
TestData = nData(NumTrain+1:end);
```

# 5: GRU �l�b�g���[�N�̍\�z
  


�u�A�v���v�^�u��I�����A�ꗗ�̒��ɂ���uDeep Network Designer�v��I�т܂��B�����āA���}�̂悤�ȃl�b�g���[�N���\�z���܂�




![image_3.png](README_images/image_3.png)




�R�}���h���C���̏ꍇ�A���̂悤�ɏ����Ɠ����ȃl�b�g���[�N���쐬���邱�Ƃ��ł��܂��BDeep Network Designer �ɂ͍쐬�����l�b�g���[�N�ɑ΂��� MATLAB �R�[�h�𐶐�����@�\���񋟂���Ă���܂�



```matlab:Code
layers_1 = [ ...
    sequenceInputLayer(1)
    gruLayer(200)
    fullyConnectedLayer(1)
    regressionLayer];
```

# 6: �w�K�I�v�V�����̐ݒ�

```matlab:Code
options = trainingOptions('adam', ...
    'MaxEpochs',500, ...
    'GradientThreshold',1, ...
    'InitialLearnRate',0.005, ...
    'LearnRateSchedule','piecewise', ...
    'LearnRateDropPeriod',125, ...
    'LearnRateDropFactor',0.2, ...
    'Verbose',0, ...
    'Plots','training-progress');
```

  
# 7: ���O�w�K�ς݊w�K����쐬

```matlab:Code
%% Train GRU network
XTrain = TrainData(1:end-1)'; % transpose to make the vector horizontally
YTrain = TrainData(2:end)';

% tic;
net = trainNetwork(XTrain,YTrain,layers_1,options);
```


![figure_1.png](README_images/figure_1.png)


```matlab:Code
% toc; % Elapsed time is 178.584624 seconds
save('TrainedGRUNetwork', 'net')
net0 = net; % backup the trained network for Simulink demo
```

# 8: 50�����̔����������ڂ��ċA�I�ɗ\�z (MATLAB��)


Simulink �ɂ��V�~�����[�V�����̎��s�̑O�ɁAMATLAB ��ōċA�I�ɔ��������𐄒肵�܂��BSeriesNetwork�N���X�unet�v�̃��\�b�h�̂P�upredictAndUpdateState�v�́A����l���g���ăl�b�g���[�N���X�V����@�\��񋟂��܂�



```matlab:Code
NumTest = numel(TestData) - 1;
YPred = zeros(1, NumTest);

for n = 1:NumTest
    [net, YPred(n)] = predictAndUpdateState(net, TestData(n));
end

RMSE = sqrt(mean((YPred-TestData(2:end)').^2));
disp(['RMSE = ', num2str(RMSE)])
```


```text:Output
RMSE = 0.3298
```

  


����l�̎��n��uYPred�v�́A���K�����ꂽ�l�ɂȂ��Ă��܂��̂Ō��̃X�P�[���ɖ߂��urYPred�v�Ƃ��܂�



```matlab:Code
rYPred = sigma .* YPred + mu;
```

# 9: MATLAB�ɂ�鐄�ڗ\�z�̉���

```matlab:Code
figure; axH = axes;
pH1 = plot(axH, T, RawData);
hold on
pH2 = plot(axH, T(NumTrain+1:end),[RawData(NumTrain+1) rYPred],".-");
fH = fill(axH, T([1,NumTrain, NumTrain, 1]), [repmat(axH.YLim(1),1,2), repmat(axH.YLim(2),1,2)], [.9, .9, .9]);
uistack(fH,"bottom")
hold off
xlabel("Month")
ylabel("Cases")
title("Forecast the number of chickenpox cases using GRU")
legend([pH1, pH2, fH], ["Observation" "Forecast", "Training Period"])
```


![figure_2.png](README_images/figure_2.png)

# 10: 50�����̔����������ڂ��ċA�I�ɗ\�z (Simulink��)


���ɁA�����ȍċA�I����� Simulink ��Ŏ��{���܂��B���O�w�K�ς݂̃l�b�g���[�N�����������ASimulink ���f���֎��n��f�[�^����͕ϐ��Ƃ��ēn�����߂Ɏ����f�[�^�z��uTimeData�v����ѕ������n��z��uSimInVariables�v���쐬���܂�




![image_4.png](README_images/image_4.png)



```matlab:Code
net = net0; % reset the pre-trained network

TimeData = T(NumTrain+1:end-1).';
SimInVariables = [TestData(1:end-1), TestData(2:end)];
```



�����āA�uset_param�v�R�}���h���g���ă\���o�[��X�e�b�v�T�C�Y�Ȃǂ�ݒ肵����Ɂurun�v�����s����ƃV�~�����[�V�������J�n���܂�




![image_5.png](README_images/image_5.png)



```matlab:Code
mdl = 'recursive_gru_update';
open_system(mdl);

set_param(mdl, 'Solver', 'FixedStepAuto');
set_param(mdl, 'FixedStep', '1');
set_param(mdl, 'StartTime', num2str(TimeData(1)));
set_param(mdl, 'StopTime',  num2str(TimeData(end)));

% Run simulation
out = sim(mdl);
```

# 11: Simulink �ɂ�鐄�ڗ\�z�̉���


MATLAB �̎��Ɠ��l�� Simulink ���f�� ��������ڗ\�����������܂�



```matlab:Code
figure; axH2 = axes;
pH3 = plot(axH2, T, RawData);
hold on
pH4 = plot(axH2, T(NumTrain+1:end),[RawData(NumTrain+1); out.ScopeOut.signals(1).values],".-");
fH2= fill(axH2, T([1,NumTrain, NumTrain, 1]), [repmat(axH2.YLim(1),1,2), repmat(axH2.YLim(2),1,2)], [.9, .9, .9]);
uistack(fH2,"bottom")
hold off
xlabel("Month")
ylabel("Cases")
title("Forecast the number of chickenpox cases using GRU")
legend([pH3, pH4, fH2], ["Observation" "Forecast", "Training Period"])
```


![figure_3.png](README_images/figure_3.png)

# 12: MATLAB �� Simulink �̈�v������

```matlab:Code
isequal(out.ScopeOut.signals(1).values, rYPred.')
```


```text:Output
ans = 
   1

```

