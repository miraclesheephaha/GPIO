## General Purpose Input and Output  
> GPIO就是晶片(CPU、微控制器、樹莓派)對外的多功能數位開關。
> 特點是沒有固定功能。在硬體設計好後，工程師可以透過程式來決定這跟針腳的功能，如接收訊號、控制開關。

 1. 輸入模式：當作傳感器或開關。晶片讀取該腳位的電位高低，用來得知外部世界的狀態。
 2. 輸出模式：當作控制開關。晶片送出高電位或低電位，來驅動外部元件。
 3. 複用功能，由於晶片的針腳數量有限，很多GPIO接腳不只可以當普通開關，還可以設定成專門的通訊模型。

### Overview
> The PCH General Purpose Input/Output signals are grouped into multiple groups (e.g., GPP_A, GPP_B, etc.) and are powered by either the PCH Primary well or Deep Sleep well.

1. PCH(Platform Controller Hub)：南橋晶片，負責處理I/O運算。
2. Primary Well：主要電源區域，在系統處於正常運作狀態(S0)時供電。
3. Deep Sleep Well(DSW)：深層睡眠電源區域，即使在系統進入極低功號睡眠狀態(S4/S5)時，為了維持某些喚醒功能，該區域仍可能維持供電。

> *翻譯* PCH 的通用型輸入輸出（GPIO）訊號被劃分為多個群組（例如：GPP_A、GPP_B 等），並由 PCH 的主要電源區域（Primary well）或深層睡眠電源區域（Deep Sleep well）進行供電。

