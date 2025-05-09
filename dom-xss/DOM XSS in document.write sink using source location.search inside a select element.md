## 1️⃣ معلومات عامة

|العنصر|التفاصيل|
|---|---|
|**رابط اللاب**|[DOM XSS in `document.write` sink using source `location.search` inside a select element](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element)|
|**تاريخ الحل**|29 / 4 / 2025|
|**نوع الثغرة**|DOM Cross-Site Scripting (XSS)|

---

## 2️⃣ فهم سيناريو اللاب

- الموقع يعرض صفحة المنتج بالشكل التالي:

![Pasted image](../images/Pasted%20image%2020250429010034.png)

- الفكرة أن الموقع يجلب المنتج بناءً على الباراميتر الموجود في الرابط وهو `productId=1`.


---

### 🔍 تحليل الكود المصدر (Source Code)

- أثناء فحص الكود المصدر وجدت الكود التالي:


```javascript
var stores = ["London", "Paris", "Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');

document.write('<select name="storeId">');

if (store) {
  document.write('<option selected>' + store + '</option>');
}

for (var i = 0; i < stores.length; i++) {
  if (stores[i] === store) {
    continue;
  }
  document.write('<option>' + stores[i] + '</option>');
}

document.write('</select>');
```

#### شرح الكود:

- هناك مصفوفة تحتوي على أسماء مخازن ثابتة: `London`, `Paris`, `Milan`.

- يتم استخراج قيمة **`storeId`** من **`location.search`** (أي من رابط الصفحة).

- يتم إنشاء عنصر **`<select>`** جديد باستخدام `document.write`.

- إذا كانت هناك قيمة في `storeId`، تتم إضافتها داخل `<option selected>`.

- بقية المخازن تتم إضافتها كخيارات أخرى داخل نفس `<select>`.


---

### تجربة:

- قمت بتجربة وضع كلمة عشوائية كقيمة لـ `storeId` مثل:


```
product?productId=1&storeId=z3rob
```

- النتيجة:

    ![Screenshot](../images/Screenshot%202025-04-29%20010617.png)

- كما هو ظاهر بالصورة: الكلمة `z3rob` ظهرت داخل القائمة المنسدلة وتم حقنها مباشرة بدون أي فحص.


---

## 3️⃣ فكرة الهجوم (Exploit Idea)

- قمت بإرسال هذا الرابط:


```
product?productId=1&storeId="></select><img src=1 onerror=alert(1)>
```

### ماذا يحدث هنا بالضبط؟

#### الخطوة 1: كسر البناء الأساسي

- بداية قيمة `storeId` بـ `">` تقوم **بإغلاق** الوسم المفتوح `<option>`.


#### الخطوة 2: كسر عنصر `<select>`

- بعد ذلك يتم إرسال `</select>` لإغلاق عنصر القائمة المنسدلة `<select>` بالكامل.


#### الخطوة 3: حقن عنصر جديد

- يتم بعدها إنشاء عنصر جديد `<img>` مع خاصية `onerror` التي تحتوي على `alert(1)`.

- بما أن الصورة غير موجودة (src=1)، يتم تنفيذ الكود داخل `onerror`.
    
    ![Pasted image](../images/Pasted%20image%2020250429011623.png)
    

### النتيجة:

- بمجرد زيارة الرابط، يتم تنفيذ كود **JavaScript** (ظهور `alert(1)`) مما يؤكد وجود ثغرة **DOM XSS** بنجاح.


---

# ✅ ملاحظات إضافية:

- الثغرة موجودة لأن الكود يدمج مدخلات المستخدم مباشرةً داخل HTML عبر `document.write` بدون أي فحص أو ترميز (Sanitization / Escaping).

- استخدام `document.write` خطير للغاية ويُعتبر من **الـ Bad Practices** في البرمجة الحديثة.

- من المفترض بدلًا من `document.write` أن يتم استخدام تقنيات أكثر أمانًا مثل `textContent` أو إنشاء عناصر DOM بشكل مهيكل عبر `createElement`.
