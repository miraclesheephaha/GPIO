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

> SCI and IOxAPIC interrupt capability is available on all GPIOs. NMI and SMI capability is available on selected GPIOs only.

1. NMI(Non-Maskable Interrupt) 特點：強制執行CPU無法透過設定暫存器來忽略它。常見用途：嚴重的硬體錯誤（如記憶體 ECC 錯誤）、系統當機時的強制調試（Watchdog）、電壓異常告警
2. SMI(System Management Interrupt) 常見用途：電源管理、風扇控制、BIOS 模擬（如舊式 USB 模擬 PS/2）。
3. SCI(System Control Interrupt) ACPI專用。通常由作業系統的ACPI驅動程式處理 常見用途：筆電蓋子合上、按下電源鍵（短按）、電池電量改變等電源事件。

#### Miscellaneous Configuration  
GPIO Group to GPE_DW0 assignment encoding (GPE0_DW0): This register assigns a specific GPIO Group to the ACPI GPE0[31:0].
> 這是一個關於 ACPI 喚醒機制（Wake Mechanism） 的核心設定。它的主要作用是建立 「GPIO 訊號」 與 「作業系統（OS）事件」 之間的橋樑。
> 簡單來說：它決定了當你按下機殼上的某個按鍵或蓋上螢幕時，PCH 要如何通知 Windows（或是其他 OS）去執行對應的 ACPI _Lxx 或 _Exx 方法。
> **範例** 0h = GPP_A[23:0] mapped to GPE[23:0]
> 當你設定為 0h 時，硬體會進行以下映射：  
* 實體訊號：GPP_A 群組的第 0 號到第 23 號腳位。
* ACPI 位元：對應到 GPE0 暫存器的第 0 到第 23 位元。
* 結果：如果 GPP_A5 有訊號跳動，OS 會收到一個 GPE[5] 的事件。

GPIO Dynamic Local Clock Gating Enable (GPDLCGEN): Specifies whether the GPIO Community should perform local clock gating.  
0 = Disable dynamic local clock gating.  
1 = Enable dynamic local clock gating.  
> GPDPCGEN (Partition Clock Gating) 非常相似，但層次（Granularity）更細。如果 Partition Clock Gating 是「關掉整個辦公室的總電源」，那麼 Local Clock Gating (GPDLCGEN) 就是「關掉個別座位的電燈」。

#### Pad Configuration  
<img width="660" height="167" alt="image" src="https://github.com/user-attachments/assets/9ef48150-247a-4e91-b13f-51cebcb5bc04" />  

這是一個關於 「記憶力」 的設定。Pad Reset Config (PADRSTCFG) 的核心功能是決定：當電腦發生重置（Reset）時，這根 GPIO 針腳的設定值（如輸入/輸出、高低電位、電阻）是否要「被打回原形」？  
在硬體設計中，這決定了訊號在開機、重新啟動或睡眠喚醒過程中的穩定性。  

1. 為什麼需要不同的 Reset 訊號？
> 在電腦運作中，Reset 分為很多等級。有些只是軟體重啟，有些是整台電腦斷電重來。
* 00 = RSMRST# (Resume Reset)
  * 等級：最高等級（最不容易被觸發）。




