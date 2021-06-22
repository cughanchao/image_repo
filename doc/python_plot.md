# python note：plot figure





### plot_txt.py

从txt文件抓取数据绘制曲线

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os, sys
import numpy as np
import matplotlib.pyplot as plt
import math

dt = []
speed = []

fileName = '/home/hanchao/Desktop/log.txt'
with open(fileName, 'r') as filePtr:
    for line in filePtr.readlines():
        print(line)
        lineStr = line.split(' ') # space separator
        if 'SetSteerCmd:' in lineStr and 'dt' in lineStr:
            dt.append(float(lineStr[-2]))  # last word is unit
        if 'SetSteerCmd:' in lineStr and 'speed' in lineStr:
            speed.append(float(linestr[-2]))
            
plt.figure(1)
plt.plot(dt, speed, label='speed $m/s$')
plt.legend()
plt.grid()

plt.show()  
```



### plot_model.py

从bag提取数据，并保存为csv文件

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os, sys
import numpy as np
import matplotlib.pyplot as plt
import math
import rosbag
import csv

reload(sys)
sys.setdefaultencoding('utf-8')

if __name__ == "__main__":
    
    timeCtrl = []
    steerCmd = []
    timeChassis = []
    steerCmdChassis = []
    
    bagPath = '/home/hanchao/Desktop/'
	bagList = os.listdir(bagPath)
    bagList.sort()
    bagName = []
    
    for i in range(len(bagList)):
        if bagList[i].endswith('.bag'):
            bagName.append(os.path.join(bagPath, bagList[i]))
    # print(bagName)
    for bagNameTmp in bagName:
        with rosbag.Bag(bagNameTmp) as bagPtr:
            for topic, msg, t in bagPtr.read_messages():
                if topic == '/vehicle_control' or topic == 'vehicle_control':
                    time = msg.header.stamp.secs + msg.header.stamp.nses / 1e9
                    timeCtrl.append(time)
                    steerCmd.append(msg.steeringAngle)
                if topic == '/chassis_command' or topic == 'chassis_command':
                    time = msg.header.stamp.secs + msg.header.stamp.nses / 1e9
                    timeChassis.append(time)
                    steerCmdChassis.append(msg.steerAngleCmd)
    timeTmp = [timeCtrl[0], timeChassis[0]]
    timeStart = min(timeTmp)(
    t1 = np.array(timeCtrl) - timeStart
    t2 = np.array(timeChassis) - timeStart
    
    csvHeader = ['time', 'csvSteerCmd', 'csvSteerChassisCmd']
    csvTime = t2
    csvSteerCmd = np.interp(csvTime, t1, steerCmd)
    csvSteerChassisCmd = steerChassisCmd

    dataStart = 10.0 # s
    dataEnd = 100.0 #s
    dataStartIndex = csvTime.tolist().index(min(csvTime, key=lambda x : abs(x - dataStart)))
    dataEndIndex = csvTime.tolist().index(min(csvTime, key=lambda x : abs(x - dataEnd)))
    
    csvFileBaseName = 'data.csv'
    csvFileName = os.path.join(bagPath, csvFileBaseName)
    print('csvFileName = %s' % csvFileName)
    with open(csvFileName, 'w) as csrFp:
        dataWrite = csv.writer(csvFp)
        dataWrite.writerow(csvHeader)
        for i in range(len(dataStartIndex, dataEndIndex):
            dataTmp = (csvTime[i], csvSteerCmd[i], csvSteerChassisCmd[i])
            dataWrite.writerow(dataTmp)
                       
	plt.figure(1)                       
    plt.plot(csvTime, csvSteerCmd, label='steer cmd')
    plt.plot(csvTime, csvSteerChassisCmd, label='chassis steer cmd')
    plt.legend()                       
    plt.grid()
                       
	plt.show()                       
```



### plot_model_check.py

读取csv数据，并绘制曲线

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os, sys
import numpy as np
import matplotlib.pyplot as plt
import math
import rosbag
import csv

reload(sys)
sys.setdefaultencoding('utf-8')

if __name__ == "__main__":
    
    time = []
    steerCmd = []
    steerCmdChassis = []
    
    csvPath = '/home/hanchao/Desktpo/'
    csvFileBaseName = 'data.csv'
    csvFileName = os.path.join(csvPath, csvFileBaseName)
    print(csvFileName)
    with open(csvFileName, 'r') as csvFp:
        csvReader = csv.reader(csvFp)
        csvTitle = next(csvReader)
        print(csvTitle)
        for lineData in csvReader:
            time.append(float(lineData[0]))
            steerCmd.append(float(lineData[1]))
            steerCmdChassis.append(float(lineData[2]))

	plt.figure(1)                       
    plt.plot(time, steerCmd, label='steer cmd')
    plt.plot(time, steerChassisCmd, label='chassis steer cmd')
    plt.legend()                       
    plt.grid()
                       
	plt.show()              
            
```



### C++读写csv

c++ 读写csv文件，用法跟读写txt文本文件一样

```C++
#include <vector>
#include <string>
#include <fstream>
#include <sstream>

int main()
{
    std::string filePath = "/home/hanchao/Desktop/";
    
    std::string fileBaseName = "log.csv";
    std::string fileName = filePath + fileBaseName;
    std::ifstream csvFile(fileName);
    if (csvFile.is_open()) {
        std::cout << "csvFile " << fileName << " is opened successfully!";
    } else {
        std::cout << "csvFile " << fileName << " is opened failed!";
    }
    std::string line, word;
    std::vector<double> simTime, simSteerCmd;
    getline(csvFile, line); // jump the header line
    std::cout << "[HANCHAO] read csv header = " << line << "\n";
    
    while (getline(csvFile, line)) {
        std::vector<double> dateLine;
        std::istringstream ssLine(line);
        while (getline(ssLine, word, ',')) {
            dataLine.push_back(std::stof(word));
        }
        simTime.push_back(dataLine[0U]);
        simSteerCmd.push_back(dataLine(1U));
    }
    csvFile.close();
    // do something ....
    std::vector<double> yawRate, yaw;
    
    // save reponse data
    std::string saveFileBaseName("save_log.csv");
    std::string saveFileName = filePath + saveFileBaseName;
    std::cout << "[HANCHAO] saveFileName = " << saveFileName << "\n";
    std::ofstream saveData;
    saveData.open(saveFileName);
    if (saveData.is_open()) { // write csv header
        saveData << "time,yawRat,yaw\n";
    }
    
    int stepNum = simTime.size();
    for (int i = 1; i < stepNum; ++i) {
        // update repose data ...
        saveData << simTime[i] << ',' << yawRate[i] << ',' << yaw[i] << "\n";
    }
    saveData.close();
    
    return 0;
}
```

