import sys
from PyQt5.QtWidgets import *
from PyQt5 import uic

# 키생성
import numpy as np
from numpy import random

# ui 파일
form_class = uic.loadUiType("untitled.ui")[0]

class MyWindow(QMainWindow, form_class):
    
#-----------------------------------#  
    # 변수값
    check = 0               # 문자 검사
    bit10 = []              # 10bit키
    Skey1 = 0b00000000      # 서브키1
    Skey2 = 0b00000000      # 서브키2
    Enc = 0b00000000        # 암호화
    Dec = 0b00000000        # 복호화

    # 순열
    sbit10 = [2,4,1,6,3,9,0,8,7,5]  
    sbit8 = [5,2,6,3,7,4,9,8]
    sIP = [1,5,2,0,3,7,4,6]
    sIP_ = [3,0,2,4,6,1,7,5]
    sEP = [3,0,1,2,1,2,3,0]
    sP4 = [1,3,2,0]

    s0 = np.array([[[0,1],[0,0],[1,1],[1,0]],
                  [[1,1],[1,0],[0,1],[0,0]],
                  [[0,0],[1,0],[0,1],[1,1]],
                  [[1,1],[0,1],[1,1],[1,0]]])
    s1 = np.array([[[0,0],[0,1],[1,0],[1,1]],
                  [[1,0],[0,0],[0,1],[1,1]],
                  [[1,1],[0,0],[0,1],[0,0]],
                  [[1,0],[0,1],[0,0],[1,1]]])
#-----------------------------------#
    
    def __init__(self):
        super().__init__()
        self.setupUi(self)
        self.pushButton.clicked.connect(self.btn_clicked)   # 서브키 생성
        self.pushButton_2.clicked.connect(self.btn_clicked2) # 암호화
        self.pushButton_3.clicked.connect(self.btn_clicked3) # 복호화
        
    # 10bit key    
    def btn_clicked(self):
        text = self.lineEdit.text()    # 입력 받기
        check = int(self.lineEdit.text())                                      #※ 입력 ※
        if 0 <= check <= 1023:
            print('hi')

        bit10 = list(format(check,'b')) # input()에서 int 입력후 format()에서 2진수형태의 str로 출력
        bit10 = list(map(int,bit10))
        bit10 = np.array([0 for i in range(10-len(bit10))] + bit10) # 빈 자리 채우기

           # SDES에서의 고정값?
        bit10 = np.array([bit10[i] for i in self.sbit10])                                 
        self.textBrowser_p10.setText(np.array2string(bit10)[1:-1])              # P10 값 출력
        
        # Shift 서브키1
        bit10 = bit10.reshape(2,5) 
        bit10 = np.stack([np.roll(bit10[0],-1), np.roll(bit10[1],-1)])         #※ LS-1 ※
        bit10 = bit10.flatten()
        self.textBrowser.setText(np.array2string(bit10)[1:-1]) 

        # 10비트 -> 8비트 치환 [2비트 버리기] K1
        global Skey1
        Skey1 = np.array([bit10[i] for i in self.sbit8])
        Skey1 = Skey1.flatten()                                                #※ 서브키1 ※
        self.textBrowser_2.setText(np.array2string(Skey1)[1:-1]) 
        self.textBrowser_5.setText(np.array2string(Skey1)[1:-1]) 
        #-----------------------------------#

        # Shift 서브키2
        bit10 = bit10.reshape(2,5)
        bit10 = np.stack([np.roll(bit10[0],-2), np.roll(bit10[1],-2)])         #※ LS-2 ※
        bit10 = bit10.flatten()
        self.textBrowser_3.setText(np.array2string(bit10)[1:-1]) 

        # 10비트 -> 8비트 치환 [2비트 버리기] K1b
        global Skey2
        Skey2 = np.array([bit10[i] for i in self.sbit8])
        Skey2 = Skey2.flatten()                                                #※ 서브키2 ※
        self.textBrowser_4.setText(np.array2string(Skey2)[1:-1]) 
        self.textBrowser_6.setText(np.array2string(Skey1)[1:-1]) 

        QMessageBox.about(self, "message", text)     # lineEdit / textBrowser ~ textBrowser_6
    
    # 암호화
    def btn_clicked2(self):
        
        global Skey1
        global Skey2
        b = ''
        QMessageBox.about(self, "message", "암호화")  # lineEdit_2 / textBrowser_7 ~ textBrowser_11
        
        for si in self.lineEdit_2.text():
            Enc = list(format(int(ord(si)),'b'))
#             Enc = list(format(int(si),'b'))                      #※ 암호화 입력 ※
            Enc = list(map(int,Enc))
            Enc = np.array([0 for i in range(8-len(Enc))] + Enc) # 빈 자리 채우기

            Enc = np.array([Enc[i] for i in self.sIP])                               #※ IP ※
            self.textBrowser_IPE.setText(np.array2string(Enc)[1:-1]) 

            EncL = Enc.reshape(2,4)[0]
            EncR = Enc.reshape(2,4)[1]

            EP = np.array([EncR[i] for i in self.sEP])
            EP = (EP+Skey1)%2  # XOR 서브키1
            EP = EP.reshape(2,4)


            P4 = np.concatenate([self.s0[2*EP[0,0]+EP[0,3], 2*EP[0,1]+EP[0,2]], self.s1[2*EP[1,0]+EP[1,3], 2*EP[1,1]+EP[1,2]]])
            P4 = np.array([P4[i] for i in self.sP4])
                                                                                      #※ fk((EncL+P4)%2, EncR)※
                                                                                      #※ SW  ※

            self.textBrowser_fk1E.setText(np.array2string(np.concatenate([(EncL+P4)%2,EncR]))[1:-1]) 
            SW = np.concatenate([EncR,(EncL+P4)%2]) # XOR, 좌우 비트 스왑
            self.textBrowser_SWE.setText(np.array2string(SW)[1:-1]) 

            SWL = SW.reshape(2,4)[0]
            SWR = SW.reshape(2,4)[1]

            EP = np.array([SWR[i] for i in self.sEP])
            EP = (EP+Skey2)%2  # XOR 서브키2
            EP = EP.reshape(2,4)


            P4 = np.concatenate([self.s0[2*EP[0,0]+EP[0,3], 2*EP[0,1]+EP[0,2]], self.s1[2*EP[1,0]+EP[1,3], 2*EP[1,1]+EP[1,2]]])
            P4 = np.array([P4[i] for i in self.sP4])                                      #※ fk(SWL+P4)%2, SWR)[UI 제작해야됨] ※
            SW = np.concatenate([(SWL+P4)%2, SWR])    
            self.textBrowser_fk2E.setText(np.array2string(SW)[1:-1]) 

            # IP-1
            a = np.array([SW[i] for i in self.sIP_])                                      #※ IP-1[UI 제작해야됨] ※
            self.textBrowser_IP1E.setText(np.array2string(a)[1:-1]) 
            a = np.array([a[0]*128,a[1]*64,a[2]*32,a[3]*16,a[4]*8,a[5]*4,a[6]*2,a[7]])
                                                               #※ 암호화
            b += chr(sum(a))
            
        self.textBrowser_17.setText(b) 
    
    # 복호화
    def btn_clicked3(self):
        
        b = ''
       # input()에서 int 입력후 format()에서 2진수형태의 str로 출력
        for si in self.lineEdit_3.text():
            Dec = list(format(int(ord(si)),'b'))                                    #※ 복호화 입력 [UI 제작해야됨] ※
            Dec = list(map(int,Dec))
            Dec = np.array([0 for i in range(8-len(Dec))] + Dec) # 빈 자리 채우기

            Dec = np.array([Dec[i] for i in self.sIP])                                    #※ IP [UI 제작해야됨] ※
            self.textBrowser_IPE_2.setText(np.array2string(Dec)[1:-1]) 

            DecL = Dec.reshape(2,4)[0]
            DecR = Dec.reshape(2,4)[1]

            EP = np.array([DecR[i] for i in self.sEP])
            EP = (EP+Skey2)%2  # XOR 서브키2
            EP = EP.reshape(2,4)

            P4 = np.concatenate([self.s0[2*EP[0,0]+EP[0,3], 2*EP[0,1]+EP[0,2]], self.s1[2*EP[1,0]+EP[1,3], 2*EP[1,1]+EP[1,2]]])
            P4 = np.array([P4[i] for i in self.sP4])
                                                                                      #※ fk((EncL+P4)%2, EncR)[UI 제작해야됨] ※
                                                                                      #※ SW [UI 제작해야됨] ※
            self.textBrowser_fk1E_2.setText(np.array2string(np.concatenate([(DecL+P4)%2,DecR]))[1:-1]) 
            SW = np.concatenate([DecR,(DecL+P4)%2]) # XOR, 좌우 비트 스왑
            self.textBrowser_SWE_2.setText(np.array2string(SW)[1:-1]) 

            SWL = SW.reshape(2,4)[0]
            SWR = SW.reshape(2,4)[1]

            EP = np.array([SWR[i] for i in self.sEP])
            EP = (EP+Skey1)%2  # XOR 서브키1
            EP = EP.reshape(2,4)


            P4 = np.concatenate([self.s0[2*EP[0,0]+EP[0,3], 2*EP[0,1]+EP[0,2]], self.s1[2*EP[1,0]+EP[1,3], 2*EP[1,1]+EP[1,2]]])
            P4 = np.array([P4[i] for i in self.sP4])                                      #※ fk(SWL+P4)%2, SWR)[UI 제작해야됨] ※
            SW = np.concatenate([(SWL+P4)%2, SWR])  # IP-1
            self.textBrowser_fk2E_2.setText(np.array2string(SW)[1:-1])


            # IP-1
            a = np.array([SW[i] for i in self.sIP_])                                      #※ IP-1[UI 제작해야됨] ※
            self.textBrowser_IP1E_2.setText(np.array2string(a)[1:-1]) 
            a = np.array([a[0]*128,a[1]*64,a[2]*32,a[3]*16,a[4]*8,a[5]*4,a[6]*2,a[7]])
            b += chr(sum(a))
            
        self.textBrowser_16.setText(b)          
        QMessageBox.about(self, "message", "복호화")  # lineEdit_3 / textBrowser12 ~ textBrowser_16

if __name__ == "__main__":
    app = QApplication(sys.argv)
    myWindow = MyWindow()
    myWindow.show()
    app.exec_()
