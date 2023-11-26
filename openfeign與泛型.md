## openfeign與泛型
---
openfeign是很強大的整合工具...可以與註冊中心搭配直接實現負載均衡,熔斷降級等


還能讓微服務之間相互調用就跟在單體式架構裡面一樣


情境:

  openfeign的client不知道server回傳的泛型是什麼導致解析失敗.
  
  例如說我定義一個feign介面:
```
      @PostMapping("/taipei/rainfall")
    public BaseResponse<rainfallContent> getRain(RainfallRequest request);
```
這裡雖然定義了一個明確的回傳類型,理論上server回傳在不出意外的情況下是可已成功的.

但是若我回傳的範型不是rainfallContent那就會解析失敗,這是當然回傳的型別跟接收的型別就不一樣了...

例如我在consumer端有做一個錯誤處理,而且剛好請求又噴錯導致被錯誤處理給回傳了一個別的泛型例如以下這樣
```
  BaseResponse<error>
```
所以基本上不能寫死...除非很確定不會回傳別的泛型
若很確定不會回傳別的泛型那理論上一開始也不用特別使用泛型了,笑死

那假設改定義成這樣

```
      @PostMapping("/taipei/rainfall")
    public BaseResponse<?> getRain(RainfallReq req);
```
這樣只要泛型中有東西,就會直接失敗.
因為泛型其實會被擦除,微服務之間是不會知道彼此類型的所以會解析失敗

---

解:

這裡提供一個懶人一點的解法

_完整的解法其實還是要自定義類解析器然後註冊給context然後在註解中調用_

**但如果不在乎在client端有點醜的話可以直接用一個map去接**

首先新建一個類去繼承 LinkedHashMap 與自己的基礎的content
**若你的response的泛型沒有對content的型別有限制的話就可以不用繼承自己的content,不過應該很少人這樣做吧**
```

@NoArgsConstructor
public class GeneralMapContent extends LinkedHashMap implements BaseContent {}
```
接著再用feign去接就好了
```
      @PostMapping("/taipei/rainfall")
    public BaseResponse<GeneralMapContent> getRain(RainfallRequest request);
```

要注意的是
  **對於client端類型還是會丟失**
  **嵌套複雜物件順序並不一定保證維持**

以上

改天再來實作客制類解析器

  
