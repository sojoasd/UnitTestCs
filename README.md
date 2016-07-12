# Unit Test

## 一、簡介

 **單元測試用意在於測試原專案的 function 是否回傳預期的值** ，例如：
```C#
public int GetNumber(int a, int b)
{
   return a + b; //回傳相加值
}
```
以上此函式是回傳相加值，那單元測試程式就要測試回傳是否為相加值，以下為單元測試寫法：
```C#
public int GetNumberTest()
{
   int a = 1;
   int b = 2;
   int sum1 = a + b;

   var obj = new SignFlowTemplateController(); //原專案 function 所屬的 class
   int sum2 = obj.GetNumber(a, b);

   //自訂判斷，並輸出自訂的錯誤訊息
   if (sum1 != sum2)
   {
       Assert.Fail("sum1 != sum2");
   }
}
```

## 二、環境建置
先安裝 VS2013 的元件「Generator UnitTest」，再來是單元測試的環境建置可分為兩種：

####  1.複製原專案的 config 檔內容
 * 優點：建置環境容易，不用擔心 function 內的 **物件相依性** 問題，其實等於複製了一個原專案
 * 缺點：若連錯資料庫，或是 config 檔內容有錯誤，雖然仍可已 run 單元測試，但得到的結果可能不如預期
 * 作法： <br>
   (1) 加入 config：測試專案 → 右鍵 → 加入 → 新增項目 → 應用程式組態檔 <br>
   (2) 複製 config：將原專案的 Web.config 內容全部複製過去測試專案的 App.config 檔 <br>
   (3) 加入參考：可以參考下圖 <br>
<img src="https://github.com/sojoasd/CsUnitTest/blob/master/Image/%E5%BB%BA%E7%BD%AE1%E7%B5%90%E6%A7%8B.JPG" width="300" height="324" /><br>
  (ps. 所謂 **相依性** 簡單來說就是若物件為 **null** 則程式就掛了)

####  2.使用 Mock 模擬任何東西
 * 優點：測試 function 的引數(物件、List、變數)可依據各種情境任意更換，不讀取資料庫資料彈性較高
 * 缺點：使用 Mock 較為麻煩，不易撰寫，且須考慮 function 內會用到的每個物件來源與相依性，很有可能會受限於原專案的程式
 * 作法：這裡先介紹加入參考這部分，如下圖，其餘做法後面會說明 <br>
<img src="https://github.com/sojoasd/CsUnitTest/blob/master/Image/%E5%BB%BA%E7%BD%AE2%E7%B5%90%E6%A7%8B.JPG" width="300" height="324" /><br>

## 三、選擇單元測試對象
此撰寫單元測試程式前，必需做好上述的環境建置，接下來我們針對 SignFlowTemplateController.GetOrgIDs() 做撰寫，步驟：

* 點選 GetOrgIDs() 的範圍建立測試專案，如下圖 <br>
<img src="https://github.com/sojoasd/CsUnitTest/blob/master/Image/%E9%81%B8%E6%93%87%E7%AF%84%E5%9C%8D.JPG" width="500" height="180" /><br>
* 建立 NUnit 測試專案，如下圖 <br>
<img src="https://github.com/sojoasd/CsUnitTest/blob/master/Image/%E9%81%B8%E6%93%87NUnit.JPG" width="500" height="310" /><br>
* 完成後，以下會分為「撰寫方法1」與「撰寫方法2」兩種分別說明

## 四、撰寫方法 1
此方法是 **複製原專案的 config 檔內容** ，程式碼如下：

```C#
//以上 using 省略，請參考範例檔

public void GetOrgIDsTest()
{
   //建立 GetOrgIDs 的引數
   var SignFlowNodeSignoffData = new List<SignFlowNodeSignoff>
   {
       new SignFlowNodeSignoff
       {
           ID = Guid.Parse("16F95B12-BCD5-4325-9542-029A7288FFF3"),
           SFNTID = Guid.Parse("3FEDD6AD-4076-4D56-8047-B5BAF1A1D318"),
           SignEmpID = Guid.Parse("88E485C0-AD68-41CF-81EF-C4547D0C8441"),
           InitUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
           InitDT = DateTime.Parse("2016-06-04 11:57:06.327"),
           ModifiedUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
           ModifiedDT = DateTime.Parse("2016-06-04 11:57:06.327")
       }
   }.AsQueryable();
   
   //建立 SignFlowTemplateController 物件
   var target = new SignFlowTemplateController();
   var result = target.GetOrgIDs(SignFlowNodeSignoffData.ToList());
   
   //自訂判斷，並輸出自訂的錯誤訊息
   if (result == string.Empty) //預期不會是空字串
   {
       Assert.Fail("string is empty");
   }
}
```
* 此處的 **target** 其實已經包含了 **db** 與 **oTypeCode** 兩個物件了，這也就是說若用 「**複製原專案的 config 檔內容**」 此方法 <br>
  則不必考慮 GetOrgIDs() 內物件的相依性問題，反正在測試專案 run 起來時都幫你建好了，不怕物件為 **null**
* 雖然解決了相依性的問題，但如果你想客製物件，使物件能依照情境來運作，那勢必不能使用此方法撰寫，就需要另外使用 **Mock** 方法了，以下將說明

## 五、撰寫方法 2
此方法是使用 **Mock** 模擬任何東西，這裡再說明一下，Mock 可以模擬任何物件、List、變數等等，目的只是為了要客製這些東西，使單元測試
更加符合情境，以下為 step by step 慢慢說明：<br>

####  1.建立 Entity 的 db 物件
因為此測試方法是沒有 config 檔的，故在 runtime 時，原專案的 db 是空的，任何 Model 都是 null，所以這邊一定要客製一個假的 Entity db 給原專案的 db 物件做接收，如下：

* 測試專案

```C#
[Test()]
public void GetOrgIDsTest()
{
   Mock<iKGDBEntities> mockEntity = new Mock<iKGDBEntities>();//建立一個虛擬的 Entity 物件
   
   var target = new SignFlowTemplateController(mockEntity.Object);//在建立 Controller 物件時一併接收假的 Entity 物件
}
```

* 原專案 - 建構子

```C#
public class SignFlowTemplateController : BaseController
{
   private iKGDBEntities db = null; //預設
   
   public SignFlowTemplateController(iKGDBEntities mockdb) //建立建構子對 db 做初始化
   {
      this.db = mockdb;
   }
}
```
此時，db 物件的 Model 都是 null，我們只需針對 GetOrgIDs() 內 db 會用到的 Model 塞資料即可，我們發現 db 只針對 **InfoFabEmpOrgs** 取資料，故這裡只需塞入 InfoFabEmpOrgs 資料即可

* 原專案 - GetOrgIDs()

```C#
public string GetOrgIDs(List<SignFlowNodeSignoff> signFlowNodeSignoffList)
{
   //......
   
   List<InfoFabEmpOrg> InfoFabEmpOrgList = db.InfoFabEmpOrgs.ToList();
   
   //......
}
```
故再將測試專案修改一下，如下

* 測試專案

```C#
[Test()]
public void GetOrgIDsTest()
{
   Mock<iKGDBEntities> mockEntity = new Mock<iKGDBEntities>();//建立一個虛擬的 Entity 物件
   
   var InfofabEmpOrgData = new List<InfoFabEmpOrg>
   {
       new InfoFabEmpOrg
           {
               ID = Guid.Parse("5FD66B6A-7BF8-4394-82D3-007DFD7C7088"),
               EmpID = Guid.Parse("01C0C2D0-526A-4E5A-B3F2-B0CB1A4E0E66"),
               OrgID = Guid.Parse("241CF569-AB49-4763-8F96-C5C1B6B0F97A"),
               OrgType = Guid.Parse("3CFEE8A5-1341-4C6E-9C61-5E0478DBF2A3"),
               SDate = DateTime.Parse("2016-02-26 00:00:00.000"),
               WorkGroupID = Guid.Parse("0BCFD6F4-D35E-4214-88F3-23C0C85F743A"),
               InitUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
               InitDT = DateTime.Parse("2016-06-04 11:57:06.327"),
               ModifiedUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
               ModifiedDT = DateTime.Parse("2016-06-04 11:57:06.327")
           },
           new InfoFabEmpOrg
           {
               ID = Guid.Parse("92AD9661-5499-4E9B-855D-01577DD7F421"),
               EmpID = Guid.Parse("AFB11BEB-A985-4FC7-8184-CC424165F610"),
               OrgID = Guid.Parse("29BE541D-D4DD-42DC-92D1-58D46C306569"),
               OrgType = Guid.Parse("3CFEE8A5-1341-4C6E-9C61-5E0478DBF2A3"),
               SDate = DateTime.Parse("2016-02-26 00:00:00.000"),
               WorkGroupID = Guid.Parse("0BCFD6F4-D35E-4214-88F3-23C0C85F743A"),
               InitUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
               InitDT = DateTime.Parse("2016-06-04 11:57:06.327"),
               ModifiedUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
               ModifiedDT = DateTime.Parse("2016-06-04 11:57:06.327")
           }
   }.AsQueryable();

   var InfoFabEmpOrgMockSet = new Mock<DbSet<InfoFabEmpOrg>>();//建立一個虛擬的 InfoFabEmpOrg Model 
   InfoFabEmpOrgMockSet.As<IQueryable<InfoFabEmpOrg>>().Setup(m => m.Provider).Returns(InfofabEmpOrgData.Provider);
   InfoFabEmpOrgMockSet.As<IQueryable<InfoFabEmpOrg>>().Setup(m => m.Expression).Returns(InfofabEmpOrgData.Expression);
   InfoFabEmpOrgMockSet.As<IQueryable<InfoFabEmpOrg>>().Setup(m => m.ElementType).Returns(InfofabEmpOrgData.ElementType);
   InfoFabEmpOrgMockSet.As<IQueryable<InfoFabEmpOrg>>().Setup(m => m.GetEnumerator()).Returns(InfofabEmpOrgData.GetEnumerator());

   mockEntity.Setup(c => c.InfoFabEmpOrgs).Returns(InfoFabEmpOrgMockSet.Object);//將資料塞入 Model 中
   
   var target = new SignFlowTemplateController(mockEntity.Object);//在建立 Controller 物件時一併接收假的 Entity 物件
}
```

此時，即可對測試專案的 GetOrgIDsTest() 範圍內點右鍵 → 偵錯測試，將斷點停留在原專案的 SignFlowTemplateController 建構子中即可查看 db 物件是否可以讀取到 InfoFabEmpOrgs 的資料，如下圖 <br>
<img src="https://github.com/sojoasd/CsUnitTest/blob/master/Image/%E6%8E%A5%E6%94%B6Entity.JPG" width="500" height="300" /><br>

但我們又發現 GetOrgIDs() 又會用到 TypeCodeUtility 物件且會用到 oTypeCode.GetSignFlowOrgTypeCodeList() 的資料，如下

* 原專案 - GetOrgIDs()

```C#
public void GetOrgIDs()
{
   //.....
   
   Guid PlanCodeID = oTypeCode.GetSignFlowOrgTypeCodeList().Where(w => w.Code == "Plan").FirstOrDefault().ID;
   
   //.....
}
```
故我們在 GetOrgIDsTest() 內也必須要建立一個 oTypeCode 物件給 SignFlowTemplateController 做接收，兩邊專案修改如下

* 測試專案

```C#
[Test()]
public void GetOrgIDsTest()
{
   //......省略 Entity 部分......
   
   var mockTypeCode = new Mock<TypeCodeUtility>();//建立一個虛擬的 oTypeCode 物件
   //模擬 GetSignFlowOrgTypeCodeList() 回傳的資料
   mockTypeCode.Setup(c => c.GetSignFlowOrgTypeCodeList()).Returns(new List<InfoFabCode>() {
       new InfoFabCode
       {
           ID = Guid.Parse("433DE584-02FD-4436-A22D-078AB61FBAC8"),
           TypeID = Guid.Parse("38EDC0AA-F3E5-48C7-9302-EC1829972401"),
           Code = "Plan",
           CodeName = "確認",
           AliasName = "確認",
           Remark = string.Empty,
           Seq = 2,
           IsDefault = false,
           OriginalCode = "Plan",
           IsSysControl = false,
           IsEnabled = true,
           InitUID = "F922A54F-BCE6-42FB-9479-EC6EE60565A5",
           InitDT = DateTime.Parse("2016-01-29 14:34:23.910"),
           ModifiedUID = "F922A54F-BCE6-42FB-9479-EC6EE60565A5",
           ModifiedDT = DateTime.Parse("2016-01-29 14:34:23.910"),
       }
   });

   var target = new SignFlowTemplateController(mockEntity.Object, mockTypeCode.Object);//多加入一個 oTypeCode 引數
}
```

* 原專案 - 建構子

```C#
public class SignFlowTemplateController : BaseController
{
   private iKGDBEntities db = null;//預設
   private TypeCodeUtility oTypeCode = null;//預設
   
   public SignFlowTemplateController(iKGDBEntities mockdb, TypeCodeUtility mocktypecode) //建立建構子對 db 做初始化
   {
      this.db = mockdb;
      this.oTypeCode = mocktypecode;
   }
}
```

此時，我們可以先執行一次偵錯測試，看看原專案的建構子是否有接收到 oTypeCode 物件與 GetSignFlowOrgTypeCodeList() 回傳的資料，但不幸的是出現了以下錯誤訊息 <br>
<img src="https://github.com/sojoasd/CsUnitTest/blob/master/Image/oTypeCode%E9%8C%AF%E8%AA%A4.JPG" width="500" height="290" /><br>

這是因為 Mock 不可以模擬「sealed class」與「non-virtual methods」，簡單來說就是 TypeCodeUtility.GetSignFlowOrgTypeCodeList() 方法宣告時不是 virtual 虛擬的，那這樣怎麼以不改原程式為主來 Mock 它呢? <br>

這邊我目前知道的方法有兩種：
 1. 重構 TypeCodeUtility，產生一個 ITypeCodeUtility (Interface)，則原專案的宣告必須改為：

* 原專案 - 建構子

```C#
public class SignFlowTemplateController : BaseController
{
   private iKGDBEntities db = null;//預設
   private ITypeCodeUtility oTypeCode = null;//用 ITypeCodeUtility 宣告，建構子用 TypeCodeUtility 實體化 oTypeCode 物件
   
   public SignFlowTemplateController(iKGDBEntities mockdb, TypeCodeUtility mocktypecode) //建立建構子對 db 做初始化
   {
      this.db = mockdb;
      this.oTypeCode = mocktypecode;
   }
}
```

 2. 直接 TypeCodeUtility.GetSignFlowOrgTypeCodeList() 方法前加「virtual」，如下：

* 原專案 - TypeCodeUtility.GetSignFlowOrgTypeCodeList()

```C#
public virtual ICollection<InfoFabCode> GetSignFlowOrgTypeCodeList()
{
   return this.GetTypeCodeList("OrgType");
}
```

如此一來就 OK 了，原專案的建構子 oTypeCode 可以正常接收到 Mock 的物件與 GetSignFlowOrgTypeCodeList() 的回傳資料了! 那接下來就是對 SignFlowTemplateController.GetOrgIDs() 方法塞入 List<SignFlowNodeSignoff> 引數了，如下圖：<br>
<img src="https://github.com/sojoasd/CsUnitTest/blob/master/Image/oTypeCode%E6%88%90%E5%8A%9F.JPG" width="500" height="240" /><br>

* 測試專案

```C#
[Test()]
public void GetOrgIDsTest()
{
   //......省略 Entity、TypeCode 部分......
   
   //建立 List<SignFlowNodeSignoff> 資料
   var SignFlowNodeSignoffData = new List<SignFlowNodeSignoff>
   {
       new SignFlowNodeSignoff
       {
           ID = Guid.Parse("16F95B12-BCD5-4325-9542-029A7288FFF3"),
           SFNTID = Guid.Parse("3FEDD6AD-4076-4D56-8047-B5BAF1A1D318"),
           SignEmpID = Guid.Parse("88E485C0-AD68-41CF-81EF-C4547D0C8441"),
           InitUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
           InitDT = DateTime.Parse("2016-06-04 11:57:06.327"),
           ModifiedUID = "0CE8300F-A623-41C3-BFE7-2706144F2DD5",
           ModifiedDT = DateTime.Parse("2016-06-04 11:57:06.327")
       }
   }.AsQueryable();
   
   var target = new SignFlowTemplateController(mockEntity.Object, mockTypeCode.Object);
   
   var result = target.GetOrgIDs(SignFlowNodeSignoffData.ToList()); //開始測試

   //自訂判斷，並輸出自訂的錯誤訊息
   if (result == string.Empty) //測試回傳的字串是否為空
   {
       Assert.Fail("string is empty");
   }
}
```

## 六、結論
由上面兩個測試方法，不難看出第二種 Mock 方法非常複雜，且可能會動到原專案的程式，故要用哪種方法來測試? 這個問題我覺得是要看原專案的程式架構，以及撰寫方式，目前有一個 SOLID 撰寫概念應該比較適合使用 Mock。
