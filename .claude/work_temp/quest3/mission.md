請幫我調查因材網 (或者類似的 PHP 專案)中，OpenID 登入與建立帳號時，是否有將時間寫入資料庫，特別是以下幾個部分：

建立帳號的時間紀錄： 請檢查處理 OpenID 建立帳號邏輯的檔案 (例如

modules/OpenID/prodb_choiceIdentity.php
 或

modules/OpenID/function_addUser.php
 中的

createUserAccount
 相關函式)。

是否有將當下時間寫入 user_info 資料表的 user_regdate (註冊日期) 欄位？
是否有將當下時間寫入 user_info 資料表的 openid_updatetime (OpenID 更新時間) 欄位？
寫入的時間格式為何？
登入的時間紀錄： 請檢查處理每次 OpenID 成功登入後重導向與寫入 Session 邏輯的檔案 (例如進入

login.php
 或是

modules/OpenID/function_OpenID.php
 中的

loginWithOpenID
 函式)。

成功登入時，是否有將登入時間寫入 user_status 資料表的 starttimestamp 欄位？
是否有將時間寫入 user_status 的 lasttimein 欄位？
關於 user_status_log 資料表：

請搜尋整個專案 (特別是 OpenID 登入與建立帳號的流程)，是否有任何 INSERT INTO user_status_log 的行為發生在「建立帳號」或「首次登入」的過程中？
或者這張表主要只負責其他統計或報表功能（例如由 php_cli 底下的排程腳本匯入）？
預期產出： 請告訴我上述三個問題的答案，並列出關鍵的程式碼片段或檔案路徑作為佐證以確認你的調查結果。
