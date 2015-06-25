# Voice-Changing
##題目發想緣起
	童年受到日本動畫名偵探柯南的影響，對於柯南的變聲器非常有興趣，因此我們組別決定嘗試製作可作聲音轉換的變聲器
實作所需材料（取得來源、價位）
	麥克風(借來的)
	自製蝴蝶結(5元)
	USB音效卡(網上購買50元)
	喇叭(借來的)
使用的現有軟體與來源
	1.Python及相關套件：Pyaudio.Numpy
	2.程式碼-Stack overflow-Python Audio Frame Pitch Change
實作過程（碰到哪些問題、如何解決）
	遇到問題：
1.SD卡容量不足：查找網路教學文件擴充
2.IOError99813.硬體不足：聲音斷點
聲音雜訊
運用哪些與課程內容中相關的技巧
	Postfix－Raspberry Pi 開機自動寄信取得IP
	ssh
組裝過程及製作教學（GPIO線材的安裝、3D列印後的組裝...）
	插入USB音效卡至raspberry pi USB孔
操作教學（做出此產品之後該如何操作）
	
1.事前軟體安裝

安裝python及其相關套件
sudo apt-get install python
sudo apt-get install python-pyaudio
sudo apt-get install python-numpy

2.USB音效卡插入後調整USB音效卡設定
音效卡插入後，輸入指令
lsusb		#確認usb音效卡是否被偵測到了
在輸入指令aplay -l 檢視目前音效裝置
一般card0為預設使用的音效卡，我們pi上所接的usb音效卡要將他變成預設的
必須更改：alsa設定檔sudo vi /etc/modprobe.d/alsa-base.conf

將
options snd-usb-audio index=-2  註解掉並存檔
之後重新開機就會調整成功
	改完後如下：
	# autoloader aliases
install sound-slot-0 /sbin/modprobe snd-card-0
install sound-slot-1 /sbin/modprobe snd-card-1
install sound-slot-2 /sbin/modprobe snd-card-2
install sound-slot-3 /sbin/modprobe snd-card-3
install sound-slot-4 /sbin/modprobe snd-card-4
install sound-slot-5 /sbin/modprobe snd-card-5
install sound-slot-6 /sbin/modprobe snd-card-6
install sound-slot-7 /sbin/modprobe snd-card-7
# Cause optional modules to be loaded above generic modules
install snd /sbin/modprobe --ignore-install snd && { /sbin/modprobe --quiet snd-ioctl32 ; /sbin/modprobe --quiet snd-seq ; : ; }
install snd-rawmidi /sbin/modprobe --ignore-install snd-rawmidi && { /sbin/modprobe --quiet snd-seq-midi ; : ; }
install snd-emu10k1 /sbin/modprobe --ignore-install snd-emu10k1 && { /sbin/modprobe --quiet snd-emu10k1-synth ; : ; }
# Keep snd-pcsp from beeing loaded as first soundcard
options snd-pcsp index=-2
# Keep snd-usb-audio from beeing loaded as first soundcard
options snd-usb-audio index=-1
# Prevent abnormal drivers from grabbing index 0
options bt87x index=-2
options cx88_alsa index=-2
options snd-atiixp-modem index=-2
options snd-intel8x0m index=-2
options snd-via82xx-modem index=-2


3.編寫聲音處理python文件
vim sound.py
# -*- coding: utf-8 -*-					#編碼格式
import pyaudio
import sys
import numpy as np
import wave
import audioop
import struct

chunk = 1024
FORMAT = pyaudio.paInt16
CHANNELS = 1
RATE = 44100
    
p = pyaudio.PyAudio()
stream = p.open(format = FORMAT,
                channels = CHANNELS,
                rate = RATE,
                input = True,
                output = True,
                frames_per_buffer = chunk)
swidth = 2
count = 0
num = int(input('What pitch '))				#輸入想要轉換的聲調
print ("* recording")
while True:
    try:							#try-except 當遇到錯誤即跳過
        if count == 1024:					#當執行1024次後會給一次停止判斷
            x = input('Do you want to continue ? If yes, please enter 1 ')
            if x == 1:
                count = 0
                print(count)
                continue
            else:
                break

        data = stream.read(chunk)
        data = np.array(wave.struct.unpack("%dh"%(len(data)/swidth), data))/2 
        data = np.fft.rfft(data)					#進行傅立葉轉換
        new_fft = np.roll(data,num*2)				#shift以進行音調變換
        if num>0:
              for i in range(0, num*2):
                    data[i] = 0
          else:
              for i in range(len(data)-num*2,len(data)-1):
                data[i] = 0
        data = np.fft.irfft(new_fft)					#反轉換
        dataout = np.array(data, dtype='int16')
        chunkout = struct.pack("%dh"%(len(dataout)), *list(dataout)) 
        stream.write(chunkout)

        count=count+1
        print(count)

    except:
        pass
        print("error")

print ("* done")
stream.stop_stream()
stream.close()
p.terminate()

工作分配表
	楊佳儒－報告呈現、資料查詢
	郭亞蓁－主程式修改（sound.py）
余美儒－設備連接設定與測試
教學文件及投影片為全組製作.
參考來源
	1.Stack overflow
	2.https://people.csail.mit.edu/hubert/pyaudio/
3.http://blog.itist.tw/2015/05/playing-audio-via-usb-sound-device-with-raspberry-pi.html
