# LoginTools

登入流程
登入 -> 請求登入結果 -> 按下政策同意後-登入行為（含註冊後自動登入）GetLobby_OK
-> 判斷卡別 -> 是否為訪客登入 手機是否驗證 -> 取得聊天資訊 ->
新增進入大廳監聽

## initWithDelegate
- 綁定delegate
- 將doLogoutThenLogin關閉

## cacheLastestLoginInfoWithAccount
- 負責重置或儲存 **帳號 密碼 登入方式**

## doLogin
### startLoginWithAccount
- 使用 **cacheLastestLoginInfoWithAccount** 設定帳密 登入方式
- 建立登入監聽
- 設定procedure裡的帳密


### startAutoLogin
- 使用 **cacheLastestLoginInfoWithAccount** 設定帳密 **空** 登入方式 **０**
- 建立登入監聽
- 設定procedure狀態：自動登入

### startLoginWithFacebookID ＆ startLoginWithAppleUserID
- 使用 **cacheLastestLoginInfoWithAccount** 設定帳號 密碼為空 登入方式
- 建立登入監聽
- 設定procedure狀態：特定id登入

### startLogoutThenLogin
- 判斷登入方式
    - commonTools_ChecknNonEmptyString 判斷帳號是否符合資格
    - 建立登入監聽
    - 使用procedure 設定使用登入方式的帳密
- 如果帳密不符資格
    - 設定 顯示alertView

## doRegist
### startRegistWithAccount
- 使用 **cacheLastestLoginInfoWithAccount** 設定帳號密碼 註冊方法
- 建立註冊監聽
- 設定procedure 註冊的帳號密碼及註冊方式

### startRegistWithFacebookID & startRegistWithAppleUserID
- 使用 **cacheLastestLoginInfoWithAccount** 設定帳號 密碼為空 註冊方法
- 建立註冊監聽
- 設定procedure 註冊的帳號 密碼為空 註冊方式

## playNow
### startPlayNow
Q:267 ->[NSUserDefaults global_RemoveKey: USER_PLAYNOW_ISSVAEED_BILL_PICTURE];
- 使用硬體裝置序號註冊

## login的Notification
### received_Notification_LoginResultInfoOK_InLoginView
- 接收login通知成功
- 移除login監聽
- 處理登入成功 使用 **handleReceiveLoginOk**

### received_Notification_LoginFail_InLoginView
- 接收login通知失敗
- 清除暫存密碼
- 移除推播
- 移除login監聽
- 處理登入失敗 使用 **handleLoginFail**

### received_Notification_ReLoginResultInfoOK_InLoginView
- 接收relogin通知成功
- 移除reLogin監聽
- 處理登入成功 使用 **handleReceiveLoginOk**

#### handleReceiveLoginOk
- 從gobal裡抓取是否登入成功
    - 是：使用 **handleLoginSuccessedWithResultInfo**
        - 下Firebase log 登入成功
    - 否：使用 **handleLoginOtherStatesWithResultInfo**
##### handleLoginSuccessedWithResultInfo
- 本地後台 下log登入結果
- 建立 Enter_WaitLobby_State 監聽
- 設定procedure狀態：enterWaitLobby

##### handleLoginOtherStatesWithResultInfo
- 關閉loading
- 顯示Alert
- 依照回傳結果分別處理
    - 後踢前的情況
        - 顯示Alert
    - 密碼錯誤六次
        - 設定procedure狀態：enterWaitLogin
        - 設定procedure stopLoginAndEnterWaitLogin
        - 建立Enter_WaitLogin監聽
        - 下Firebase log 登入失敗
    - 強制更改密碼
        - 設定procedure狀態：enterWaitLogin
        - 建立Enter_WaitLogin監聽
        - 彈出 O8App_ViewControllerForceChangePWD頁面
    - 密碼輸入錯誤太多次
        - 設定procedure狀態：enterWaitLogin
        - 設定procedure stopLoginAndEnterWaitLogin
        - 建立Enter_WaitLogin監聽
        - 顯示錯誤訊息
        - 下Firebase log 登入失敗

### received_Notification_ReLoginFail_InLoginView
- relogin失敗通知
- 清除暫存密碼
- 移除推播
- 移除reLogin監聽
- 處理登入失敗 **handleLoginFail**

#### handleLoginFail
- 下Firebase log 登入失敗
- 建立 enterwaitlogin監聽
- 顯示AlertView

### received_Notification_Enter_WaitLogin
354-382
- enter_WaitLogin通知
- 移除enter wait login監聽
- 顯示下載中
- 

## regist notification
### received_Notification_Get_Register_ResultInfo_OK
- 註冊成功
- 移除註冊監聽
- 判斷resultCode是否為１ 表示登入成功
    - 判斷是否按下政策同意
        - 下Firebase log 首次啟動註冊
    - 判斷註冊方式
        - 下Firebase log 註冊方式建立帳號成功
    - 清除為註冊廣告來源id
    - 下本地後台log 註冊
    - 建立 waitLobby監聽
    - 設定procedure enterWaitLobby
- 建立 enterWaitLogin監聽
- 設定procedure enterWaitLogin
- 顯示AlertView

### received_Notification_Register_Fail
- 註冊失敗
- 關閉 等待登入中
- 移除 register監聽
- 建立Enter_WaitLogin監聽
- 設定 procedure狀態：enterWaitLogin
- 顯示AlertView

## lobby
### received_Notification_Enter_WaitLobby
- 收到等待進入大廳通知
- 判斷是否按下政策同意
    - 下Firebase log 首次啟動登入
    - 如果帳號創建成功且登入
        - 下本地後台log login
- 移除 enter wait lobby監聽
- 建立 getLobby監聽
- 顯示loading
- 設定procedure狀態：啟動 去大廳 接收request

### received_Notification_Get_Lobby_OK
- 移除 getLobby監聽
- 下Firebase log 帳號
- 設定 firebase帳號密碼
- 下本地後台log 帳號
- 判斷手機是否驗證
- 必須符合 已綁定手機 如是訪客帳號 強迫綁定
    - 建立EnterLobby監聽
    - 設定procedure 執行接收進入大廳request
- 如未驗證 不強迫綁定
    - 建立EnterLobby監聽
    - 設定procedure 執行接收進入大廳request

## 處理各種

### received_Notification_Enter_Lobby
- 接收 enterLobby通知
- 移除 enterLobby監聽
- 執行 **handleEnterLobby**
- 關閉loading

#### handleEnterLobby
- 從global取得帳戶資料 登入資料
- 從mdtb取得判斷 accountInfo是否符合黃金或白金資格
    - stopLoginAndEnterMaryReelGameActivity
    - 彈出 小瑪麗月月抽 頁面 **MaryReelGameActivity_ViewController**
    - ReelGameActivity
        - 放入獎品圖片及資訊
- 判斷是否符合新手資格
    - enterToNoviceTeachingPage
    - 彈出新手教學頁面 **O8App_ViewControllerNovicelearning**
- 判斷是否綁定帳號 手機驗證 即給予點數
    - 符合 設定flag clearPhoneAuthFlag
    - 未符合 進入tabIndex_ChatRoomList
    - 進入 gameMenu **stopLoginAndEnterGameMenu**

## O8App_ViewControllerNovicelearning
新手教學

### viewDidLoad
- 使用 **initViewNovicelearning**
- 建立 applicationDidBecomeActive 監聽
    - 回復動畫 stepCont
        - ad gameIcon luckBag downloadVedio
    - 回復skipBtn動畫 

#### initViewNovicelearning
- 初始化
- 將步驟初始 設為0 清除教學Flag獎勵為no
- 設定圖片陣列
- 使用 **initGModeSet** & **initPicAndText** & **nextStep**

##### initGModeSet
- 取出全域資料(global)設定
- 設定是否已綁定帳號 是否驗證過
    - 設定isHaveGiftVTCard 判斷是否符合資格
- 抓出AccountSettingInfo
    - isHaveGiftVTCard ＝ YES
        - vtAry = mdtbAccountSettingInfo.data
        - 贈兩點點數
    - isHaveGiftVTCard ＝ NO
        - 贈一點點數

##### initPicAndText
- 初始化圖片與文字
- 各主背景
    - 歡迎畫面 bgView1
    - 遊戲列表 bgView2
    - 廣告 bgView3
    - 點擊遊戲 bgView4
    - 點立即玩 bgView5
    - **接續播影片**
    - 點擊左下背包 bgView6
    - 進入我的寶箱頁面 bgView9
    - 恭喜獲得 bgView10
- 設定各主背景的frame大小
- 初始化兔兔圖
    - ?bunnyView1 ～ bunnyView6
- 初始化訊息匡圖
    - ?msgBoxView1 ～ msgBoxView5
- 初始化按鈕圖
    - btnView ～ btnTitleView3
- 初始化透明按鈕
    - alphaBtn 動作：nextStep
        - stepCont+1
- 初始化JAMP按鈕
    - alphaJumpBtn 動作：jumpStep100
        - get2000Money
- 初始化結束按鈕
    - alphaEndBtn 動作：learningEnd
        - 判斷清除flag才可跳畫面 _isClearAllNoviceFlag tabIndex_GameMenu
        - 下firebase log 首次登入
- 初始化提示箭頭
    - nlArrow
- 判斷 isHaveGiftVTCard 符合才可
    - 從procedure抓取網路圖片url
    - 抓取虛擬寶卡request
        - 虛寶圖片ITLEMTYPE　(手機客端)
        - O8App_ViewNovicelearning_VTObj 抓取網路圖片
            - 設定image url·tetle·虛寶說明 (手機客端)
    - 將圖片新增到imagearray
- 教學文字
- skip button
    - 使用**imageSkip**
        - 判斷硬體 調整圖片
    - 動作：onButtonSkip
        - 下Firebase log 略過教學
        - 如果正在播放 暫停播放器 使用 **pauseThePlayer**
        - 如果已停止
            - **removePreviousPageSubviews** 移除前頁畫面元件
                - **removebtnViewSubviews** 移除btnView元件上的subView
                - **removeViewSubviews** 移除 self view 元件上 subviews
    - 有虛寶卡
        - enum_StepStatus_backLobby 回到大廳
    - 否
        - enum_StepStatus_get2000Money

### viewWillAppear
- 監聽動畫是否完成

### viewWillDisappear
- 監聽動畫是否完成

### dealloc
- 下本地後台log 結束新手教學
---

進入遊戲列表

08App_UITabBarController
setSelectedIndex -> 非自己呼叫的, 且要異動 index
_targetIndex 記住目標 index, 有可能在轉移 ViewController 過程中, 會下一拍處理