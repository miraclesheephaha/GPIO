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
  * 觸發時機：只有在主電力完全切斷（例如拔掉插頭、電池沒電）後重新供電時才會發生。
  * 用途：如果你希望某個 GPIO 在 「按下重啟鍵（Warm Reset）」 或是 「從 S3/S4 睡眠喚醒」 時，依然保持原本的設定值（不被重置），就選這個。
* 01 = Host Deep Reset (主機深層重置)
  * 等級：中等級。
  * 觸發時機：當系統進行冷開機（Cold Boot）或全球重置（Global Reset）時觸發。
  * 特別之處：手冊提到 "does not assert when in S3/S4/S5"。這意味著如果電腦只是在睡覺，這個重置訊號不會發出，GPIO 的設定會被保留。
* 10 = PLTRST# (Platform Reset)
  * 等級：最常發生的重置。
  * 觸發時機：只要電腦重新啟動（按重開機鍵、BIOS 更新完重啟），這個訊號就會生效。
  * 用途：一般的 GPIO 通常設為這個。一旦系統重啟，GPIO 就回到預設狀態，由 BIOS 重新初始化。

2. 什麼是 "Sx isolation" (Sx 隔離)？
想像一個情境：你的 GPIO 接到一個外部裝置，而在電腦進入 S3（睡眠） 狀態時，為了省電，PCH 的某些部分會斷電。
* 如果你希望在 S3 期間，這根針腳的電位不要變動（不要因為重置而亂跳，導致外部裝置誤動作），你必須選擇一個在 S3 期間不會觸發的重置訊號（例如 RSMRST#）
* 這就是所謂的 Sx Isolation：讓 GPIO 的狀態與系統的睡眠/喚醒狀態（Sx states）隔離開來，互不干擾。

<img width="668" height="84" alt="image" src="https://github.com/user-attachments/assets/13401646-79aa-4657-9f07-fc7eecc0d1cb" />  

它的核心作用是決定：當這根針腳作為「特殊功能（Native Function，如 UART、I2C、SPI）」使用時，訊號在進到控制器之前，要不要經過「處理（反向或濾波）」。  
*0=Raw RX pad state (原始狀態)*  

* 路徑：訊號從實體針腳進來，經過接收緩衝器（RX Buffer）後，直接丟給 UART 或 I2C 控制器。
* 特性：它不理會你在 GPIO 暫存器裡設定的任何「反向（RXINV）」邏輯。
* 用途：絕大多數標準協議（如 SPI）都要求最精準的原始時序，不希望中間有任何邏輯處理。

*1=Internal RX pad state (內部處理狀態)*  
* 路徑：訊號進來後，先經過 RXINV（訊號反向器）與 PreGfRXSel（可能是雜訊過濾或同步電路）的處理。
* 特性：如果你設定了 RXINV=1，那麼 Native Function 看到的訊號就會是反向過的（高變低，低變高）。
* 用途：
  * 訊號修正：如果硬體電路設計錯誤，把某個主動低位準的訊號接反了，你可以透過這個設定在晶片內部把它「翻轉」回來，而不需要改電路板。
  * 特殊協議處理：某些非標準的通訊協議可能需要這種內部的邏輯翻轉。

<img width="665" height="82" alt="image" src="https://github.com/user-attachments/assets/ac981f79-84fb-432a-aa7e-78bafcb7673c" />  

這是一個專門為硬體除錯（Hardware Debugging）與軟體模擬設計的「強制介入」功能。  
RXRAW1 就像是一個硬體內部的「假訊號產生器」。它可以讓內部邏輯（不論是 GPIO 還是 Native Function）誤以為針腳現在正處於高電位（High），而不管外面實體接腳到底接了什麼。  
* 為什麼需要 RXRAW1？ *
這個位元在量產產品中幾乎不使用，但在開發階段非常強大：

A. 模擬硬體尚未就緒的狀態  
假設你在寫一個偵測「硬碟已插入」的驅動程式，而該訊號是由某個 GPIO 的高電位來表示。但此時你手邊沒有硬碟，或者硬體電路還沒焊好。    

做法：將 RXRAW1 設為 1。  
結果：你的驅動程式會立刻讀到「硬碟已插入」的訊號，讓你可以繼續寫後面的邏輯代碼。

[GPIO ACCESS P2SB](https://medium.com/@jacksonchen_43335/bios-gpio-p2sb-70e9b829b403)







