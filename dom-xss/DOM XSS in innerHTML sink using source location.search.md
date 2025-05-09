# 🛡️ ملاحظات حل اللاب

## 1. معلومات عامة

|العنصر|التفاصيل|
|---|---|
|**الرابط**|[رابط اللاب](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink)|
|**تاريخ الحل**|26/4|
|**نوع الثغرة**|DOM-Based XSS|

---

## 2. وصف اللاب

- وجود **حقل بحث (Input field)** في الموقع.
    
- عند تجربة إدخال `"'>z3rob` ظهر أن قيمة الإدخال يتم عرضها مباشرة داخل الصفحة بدون معالجة.
    

> صورة توضيحية:  
> ![[Pasted image 20250426185935.png]]

---

## 3. فحص الكود المصدري (Source Code Review)

```javascript
// Function to display the search query in the searchMessage element
function doSearchQuery(query) {
  document.getElementById('searchMessage').innerHTML = query;
}

// Get the 'search' parameter from the URL and display it if it exists
const query = new URLSearchParams(window.location.search).get('search');
if (query) {
  doSearchQuery(query);
}
```

### ملاحظات على الكود:

- يتم قراءة باراميتر `search` من الرابط.
    
- يتم عرضه مباشرة باستخدام `innerHTML` داخل عنصر بالصفحة بدون أي فلترة أو ترميز.
    
- هذا يسمح بتنفيذ أكواد HTML وJavaScript مباشرةً داخل الصفحة.
    

---

## 4. فكرة الهجوم (Exploit Idea)

- **سبب القابلية للهجوم:** استخدام `innerHTML` بدون فلترة.
    
- **الفكرة:** استغلال `innerHTML` لحقن كود JavaScript.
    

### تجربة أولى:

- قمت بحقن:
    
    ```html
    <script>alert(1)</script>
    ```
    
- ولكن لم يتم تنفيذ السكربت (ممكن بسبب بعض السياسات مثل Content Security Policy أو بسبب طريقة تحميل الصفحة).
    

### تجربة ثانية (ناجحة):

- قمت بحقن:
    
    ```html
    <img src=x onerror=confirm()>
    ```
    
- النتيجة: تم تنفيذ الكود وظهرت رسالة تأكيد (Confirm Box).
    

---

## 5. الخلاصة

|العنصر|التفاصيل|
|---|---|
|**مكان الحقن**|داخل عنصر HTML باستخدام `innerHTML`.|
|**آلية التنفيذ**|حقن عنصر `<img>` مع خاصية `onerror` لتنفيذ الكود.|
|**النوع**|DOM-Based XSS|
|**سبب الثغرة**|إدخال المستخدم يتم تمريره إلى `innerHTML` بدون فلترة أو ترميز.|

---

