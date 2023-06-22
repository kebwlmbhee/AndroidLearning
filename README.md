# Outline
- [Outline](#outline)
- [寫 Android 的好習慣，為自己而寫](#寫-android-的好習慣為自己而寫)
- [MVC](#mvc)
- [MVVM](#mvvm)
- [SharedPreference](#sharedpreference)
- [Navigation](#navigation)

# 寫 Android 的好習慣，為自己而寫
- 將常量放至 resource 裡，避免 hardcoded，讓可維護性上升，專案越大效果越顯著
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

**使用 Navigation 可以在不同 Activity 間進行交互**