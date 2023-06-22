# Outline
- [Outline](#outline)
- [Android，為自己而寫](#android為自己而寫)
- [MVC](#mvc)
- [MVVM](#mvvm)
- [SharedPreference](#sharedpreference)
- [Navigation](#navigation)

# Android，為自己而寫
- 將常量放至 resource 裡，避免 hardcoded，讓可維護性上升，專案越大效果越顯著
- 使用 ConstraintLayout，利用圖形化介面更改介面更加高效方便
- 使用 DataBinding 前將 xml 轉換為 DataBinding 形式，轉換後仍保留原先的 layout 形式(ConstraintLayout)
- 自動縮排：Code -> Reformat code 
- 自動排序屬性(常用在 XML file)：Code -> Rearrange code
- 過於花俏的動畫不要進行使用

# MVC

[MVC](/MVC/)

**最一般的處理方式，為人垢病的就是要一直保存資料，避免 Activity 被短暫消滅後造成資料丟失**

# MVVM

[MVVM](/MVVM/)

**使用了 MVVM 後，因為 Controller 不再管理資料，Activity 短暫被消滅(返回鍵、Portrait <-> Landscape、更改語言、關閉)時，再次重啟仍會保留資料**

# SharedPreference

[SharedPreference](/SharedPreference/)

**共用的資料，可以在外部和內部進行存取**

# Navigation

[Navigation](/Navigation/)

**使用 Navigation 可以在不同 Activity 間進行交互**