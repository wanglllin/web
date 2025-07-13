## 机器人偏移量平均计算
~~~
    PROC init()
        	offset := abs(p10.trans.x - p20.trans.x)/39;
    ENDPROC
~~~
## abb机器人碰撞回退程序
~~~
MODULE MainModule
PROC Main()
WHILE TRUE DO 
	MoveJ pHome, v1000, z50, tool0;
        
Place_1:
	SearchL\Stop, di_Crash1\NegFlank, pfound1, Offs(pPlace,0,0,0), v100, tGriper\WObj:=CurWobj;//NegFlank信号取反，从pfound1开始检测到pPlace点位 
        IF di_Crash1 =0 THEN 
          GOTO Place_2;//碰撞到了，跳转到回退处理行
        ENDIF
Place_2:
      IF di_Crash1 =0 THEN //判断是否有碰撞，避免误入程序
      	 pfound1:=CRobT(\Tool:=tGriper\WObj:=CurWobj);
        	MoveL offs(pfound1,0,0,100), vMidLoad, fine, tGriper\WObj:=CurWobj;//回退100
        	Setgo go_Error, 1;//给PLC报错
        	PulseDO\PLength:=1,do_RobError;
          Stop;
          Setgo go_Error, 0;
          GOTO Place_1;//再启动时再次进入运动程序
        ENDIF
 ENDWHILE
ENDPROC
~~~
## abb机器人信号仿真
![image](https://img2024.cnblogs.com/blog/2110076/202503/2110076-20250318195936723-1047963634.png)
![image](https://img2024.cnblogs.com/blog/2110076/202503/2110076-20250318195955776-667616530.png)
![image](https://img2024.cnblogs.com/blog/2110076/202503/2110076-20250318200009102-489334025.png)
![image](https://img2024.cnblogs.com/blog/2110076/202503/2110076-20250318200032975-434488774.png)
## abb机器人中断程序
```RAPID
MODULE MainModule

      VAR intnum HPError1:= 0;
      VAR intnum HPError2:= 0;
    PROC rInitAll()
        Startmove;
        CONNECT HPError1 WITH rHP1Error;\\中断号链接中断处理程序
        ISignalDI zhongduan, 0, HPError1;\\信号链接检测到下降到0，触发中断程序
        CONNECT HPError2 WITH rHP1Error;\\此处触发同一个中断处理程序
        ISignalDI zhuanduan2, 0, HPError2;
		ISleep HPError1;\\暂时关闭中断检测
        IWatch HPError1;\\暂时关闭中断取消
    ENDPROC

    PROC Main()
        rInitAll;
        WHILE TRUE DO 
  MoveJ [[734.31,705.94,1217.50],[0.463801,-0.323519,0.803328,0.186784],[0,-1,0,0],[9E+9,9E+9,9E+9,9E+9,9E+9,9E+9]], v1000, z50, tool0;
  MoveJ [[1018.61,0.00,1217.50],[0.5,7.70773E-10,0.866025,4.45006E-10],[0,0,0,0],[9E+9,9E+9,9E+9,9E+9,9E+9,9E+9]], v1000, z50, tool0;
        ENDWHILE
    ENDPROC
   TRAP rHP1Error
     Stopmove;\\停止移动
     IDisable;
     StorePath;\\记录轨迹
     Stop;      
     RestoPath;  \\继续轨迹
     StartMove;
     IEnable; \\再次开启中断程序
  ENDTRAP  
ENDMODULE
~~~