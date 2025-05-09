# 🛡️ ملاحظات حل اللاب

## 1. معلومات عامة

| العنصر         | التفاصيل                                                                                                         |
| -------------- | ---------------------------------------------------------------------------------------------------------------- |
| **الرابط**     | [رابط اللاب](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink) |
| **تاريخ الحل** | 27/4                                                                                                             |
| **نوع الثغرة** | DOM-Based XSS                                                                                                    |

----
## 2. وصف اللاب

اللاب يحتوي على ثغرة DOM-Based XSS في صفحة إرسال الآراء (Submit Feedback). يتم استخدام مكتبة jQuery لتحديد عنصر رابط (Anchor) وتغيير سمة `href` الخاصة به بناءً على بيانات من `location.search` (الجزء الخاص بالبارامترات في الرابط).

الهدف: جعل زر "Back" ينفذ `alert(document.cookie)`.

![Screenshot](../images/Screenshot%202025-04-27%20011030.png)

هتلاحظ من الصورة اننا غيرنا المسار ف اتحط داخل ال `href`
```html
<a id="backLink" href="z3rob">Back</a>
```

و لما تيجي بالماوس على ال `Back` هتلاقي الكلمة ظهرت ف المسار تحت

---
## 3. فحص الكود المصدري (Source Code Review)

```html
<a id="backLink">Back</a>

<script>
  $(function() {
    const returnPath = (new URLSearchParams(window.location.search)).get('returnPath');
    $('#backLink').attr("href", returnPath);
  });
</script>
```
### ملاحظات على الكود:
- يتم قراءة البارامتر `returnPath` من الرابط باستخدام `location.search`.
- يتم استخدام دالة `$` من jQuery لتحديد عنصر `<a>` وتحديث سمة `href` بقيمة `returnPath` مباشرةً بدون تعقيم أو ترميز.

---
## 4. فكرة الهجوم (Exploit Idea)

### الخطوات التفصيلية:
1. **تحديد البارامتر الضعيف**: البارامتر `returnPath` يتحكم مباشرةً في `href` الخاص برابط "Back".
2. **حقن JavaScript URI**: استبدال قيمة `returnPath` بـ `javascript:alert(document.cookie)` لتنفيذ الكود عند النقر على الرابط.
3. **التنفيذ**: إرسال الرابط المُحقن للضحية أو تحديث الصفحة يدويًا ثم النقر على "Back".

### مثال الرابط المُهاجم:
```
https://vulnerable-lab.com/feedback?returnPath=javascript:alert(document.cookie)
```


---
## 6. الوقاية (Prevention)

- **تعقيم المدخلات**: استخدام دوال مثل `encodeURIComponent()` عند إدخال بيانات المستخدم في السمات.
- **السمات الآمنة**: تجنب استخدام `javascript:` في السمات الديناميكية مثل `href`.
- **Content Security Policy (CSP)**: تطبيق سياسة أمان تمنع تنفيذ الكود غير الموثوق.