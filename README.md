# whatsapp-chat-wordcloud
Make a figure (wordcloud) from whatsapp chat

## Install packages
```python
!pip install --user -U nltk
!pip install wordcloud
!pip install arabic_reshaper
!pip install python-bidi
```

## Imports
```python
import re
import os
import sys
import fileinput
import nltk
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
import pandas as pd
import codecs
import string
import matplotlib.pyplot as plt
from wordcloud import WordCloud
import arabic_reshaper
from bidi.algorithm import get_display
```

## Downloads
```python
nltk.download('stopwords')
nltk.download('punkt')
```

## Remove emoji
```python
# Remove emoji
def remove_emoji(inputString):
    emoji_pattern = re.compile("["
        u"\U0001F600-\U0001F64F"  # emoticons
        u"\U0001F300-\U0001F5FF"  # symbols & pictographs
        u"\U0001F680-\U0001F6FF"  # transport & map symbols
        u"\U0001F1E0-\U0001F1FF"  # flags (iOS)
        u"\U00002702-\U000027B0"
        u"\U000024C2-\U0001F251"
        u"\U0001f926-\U0001f937"
        u'\U00010000-\U0010ffff'
        u"\u200d"
        u"\u2640-\u2642"
        u"\u2600-\u2B55"
        u"\u23cf"
        u"\u23e9"
        u"\u231a"
        u"\u3030"
        u"\ufe0f"
        u"\u2069"
        u"\u2066"
        u"\u200c"
        u"\u2068"
        u"\u2067"
        "]+", flags=re.UNICODE)
    return emoji_pattern.sub(r'', inputString)
```

## Remove garbage character (punctuations, english characters, numbers)
```python
def remove_garbage(inputString):
    garbage_character = "«»!""#$%&'()*+,-./:;<=>?@[\]^_`{|}~،,؛0123456789؟abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    no_garbage = ""
    for char in inputString:
        if char not in garbage_character:
            no_garbage = no_garbage + char
        
    return no_garbage
```

## Persian stopwords
```python
stoplist = [ "﷼", "علاقه مند", "یاب", "گیر", "گوی", "کن", "شو", "دیگران", "دار", "خواه", "توان", "باش", "آی", "آور", "یکی", "یکهزار", "یکسال", "یکریز", "یکدیگر", "یک کمی", "یک کم", "یک چیزی", "یک جوری", "یک", "یواش یواش", "یواش", "یو", "یه", "یكی", "یكپارچه", "یكنواخت", "یكطرفه", "یكسری", "یكسره", "یكسال", "یكزمان", "یكریز", "یكدیگر", "یكدیر", "یكدم", "یكجوری", "یكجور", "یكجانبه", "یكجا", "یكباره", "یكبار", "یكایك", "یك", "یقیناً", "یقینا", "یعنی", "یشتری", "یشتر", "یش", "یست", "یری", "یرونِ", "یرد", "یافتیم", "یافتید", "یافتی", "یافته", "یافتن", "یافتم", "یافت", "یارب", "یاد", "یابیم", "یابید", "یابی", "یابند", "یابم", "یابد", "یاب", "یااینكه", "یاانكه", "یااز", "یا", "ی", "گیریم", "گیرید", "گیری", "گیرند", "گیرم", "گیرد", "گیر", "گيري", "گيرد", "گوییم", "گویید", "گویی", "گویند", "گویم", "گوید", "گویان", "گویا", "گوی", "گويند", "گويد", "گونه", "گو", "گهگاهی", "گهگاه", "گه", "گمان", "گفتیم", "گفتید", "گفتی", "گفته", "گفتند", "گفتن", "گفتم", "گفت", "گرچه", "گروهی", "گروهي", "گرنه", "گرمی", "گرفتیم", "گرفتید", "گرفتی", "گرفته", "گرفتند", "گرفتن", "گرفتم", "گرفتارند", "گرفت", "گردید", "گردند", "گردد", "گر", "گذشته", "گذاشتیم", "گذاشته", "گذاشتن", "گذاری", "گذاري", "گذاران", "گاهی", "گاه", "گ", "کی", "کَی", "کو", "که", "کنیم", "کنید", "کنی", "کنونی", "کنون", "کنه", "کننده", "کنند", "کنن", "کنم", "کند", "کنایه‌ای", "کنان", "کنارِ", "کنارش", "کنار", "کن", "کمی", "کمک", "کمتری", "کمتره", "کمتر", "کماکان", "کمااینکه", "کم کم", "کم", "کلیه", "کلی", "کلا", "کل", "کسی", "کسانی", "کس", "کردین", "کردیم", "کردید", "کردی", "کرده", "کردند", "کردن", "کردم", "کرد", "کرات", "کدام", "کجاست", "کجا", "کتبا", "کاملاً", "کاملا", "کامل", "کاشکی", "کاش", "کارند", "کار", "کا", "ک", "ژ", "چیه", "چیست", "چیزیست", "چیزی", "چیزهاست", "چیز", "چی", "چگونه", "چکار", "چيزي", "چونه", "چون", "چو", "چهارهزار", "چهار", "چه طور", "چه بسا", "چه", "چنین", "چنين", "چندین", "چنده", "چندمین", "چندماهه", "چندروزه", "چندانی", "چندان", "چند روزه", "چند", "چنانکه", "چنانچه", "چنانكه", "چنان", "چكار", "چقدر", "چطور", "چشم بسته", "چشم", "چراكه", "چرا که", "چرا", "چته", "چت", "چاپلوسانه", "چاله", "چارده", "چ", "پیگیر", "پیوسته", "پیشِ", "پیششون", "پیشتر", "پیشاپیش", "پیش", "پیرامون", "پیدرپی", "پیداست", "پیدا", "پیامبرانه", "پی درپی", "پی", "پيش", "پهن شده", "پهن", "په", "پنهان", "پنج", "پشیمونی", "پشتوانه اند", "پشتوانه", "پشت", "پس فردا", "پس از", "پس", "پریروز", "پروردگارا", "پرسان", "پراز", "پذیرند", "پذیر", "پدیده", "پدرپی", "پدرانه", "پایین ترند", "پایین", "پاعینِ", "پاره‌ای", "پاره", "پارسایانه", "پارسال", "پ", "٪", "يكي", "يكديگر", "يك", "يابد", "يا", "ویژه", "ویند", "وید", "ویا", "وی", "وگو", "وگرنه", "وگرد", "وگر", "وي", "وهمین", "ونن", "ولیكن", "ولی", "وقتیکه", "وقتیكه", "وقتی که", "وقتی", "وقتي", "وضوح", "وضع", "وسطِ", "وزو", "ورای", "ورا", "ور", "وده", "ودند", "ودن", "ود", "وحشت زده", "وحشت", "وجودیكه", "وجود", "وجه", "وجدانا", "وبرای", "وای", "واما", "واقفند", "واقعی", "واقعاً", "واقعا", "واقع", "واضحتر", "واضح", "واسه", "واسش", "وارد", "وابسته اند", "وابسته", "و لا غیر", "و", "هیچ‌جور", "هیچیك", "هیچی", "هیچگونه", "هیچگاه", "هیچکس", "هیچکدام", "هیچوقت", "هیچكس", "هیچكدام", "هیچجور", "هیچ گاه", "هیچ جور", "هیچ", "هی", "هيچ", "هوی", "هوشیارانه", "هوشمندانه", "هنگامیكه", "هنگامی که", "هنگامی", "هنگامِ", "هنگام", "هنوز", "هنامی", "هنامِ", "هنام", "همینكه", "همینطوریكه", "همینطوری", "همینطوركه", "همینطور", "همین که", "همین", "همیشگی", "همیشه", "همی", "همگی", "همگان", "همگام", "همچین", "همچون", "همچنین", "همچنين", "همچنانكه", "همچنان که", "همچنان", "همين", "همون", "همواره", "همه‌مون", "همه‌اش", "همهٌ", "همه مون", "همه شان", "همه ساله", "همه روزه", "همه", "همنوا", "هممون", "همكارانه", "همسوبا", "همسو", "همزمان", "همدیگر", "همدلانه", "همخوان", "هماهنگتر", "همانی", "همانها", "همانند", "همانقدر", "همانطوریكه", "همانطوری", "همانطوركه", "همانطور", "همانا", "همان گونه که", "همان طور که", "همان", "هم اینک", "هم اکنون", "هم", "هق هق کنان", "هق", "هفت", "هشیارانه", "هشان", "هسن", "هستیم", "هستید", "هستی", "هستيم", "هستند", "هستن", "هستم", "هستش", "هستا", "هست", "هزارها", "هزار", "هرگونه", "هرگز", "هرگاه", "هرکی", "هرکس", "هرچه", "هرچند", "هرچقدر", "هروقت", "هركه", "هركسی", "هركس", "هركدام", "هرقدر", "هرساله", "هرزچندگاهی", "هرز", "هرحال", "هرجا", "هرانچه", "هرازچندگاهی", "هر چه", "هر چند که", "هر چند", "هر از گاهی", "هر", "هدف", "هترین", "هبچ", "هایی", "هایت", "های", "هايي", "هاي", "هانتا", "هان", "هامو", "هام رو", "هاش", "هاست", "ها", "ه", "نیمی", "نیم", "نیك", "نیستیم", "نیستند", "نیستم", "نیست", "نیز", "نیازمندند", "نیازمندانه", "نگو", "نگاه", "نکنیم", "نکنید", "نکنی", "نکنند", "نکنم", "نکند", "نکن", "نکرده", "نکردن", "نيست", "نيز", "نومید", "نوعی", "نوعي", "نوعاً", "نوعا", "نوع", "نوشت", "نود", "نواورانه", "نهایتاً", "نهایتا", "نهایت", "نهان", "نه تنها", "نه", "نمی‌کند", "نمی‌شود", "نمیگیرند", "نمیمونید", "نمیكنند", "نمیكنن", "نمیشه", "نمیروم", "نمیتونه", "نمیبرد", "نمیاورم", "نمی", "نمي", "نموده", "نمایش", "نماید", "نمايد", "نكنه", "نكنند", "نكرده", "نقادانه", "نفهمی", "نفرند", "نفر", "نظیر", "نظير", "نظربه", "نظرا", "نشون", "نشه", "نشده", "نشان", "نسبتا", "نسبت", "نزدیکِ", "نزدیک", "نزدیم", "نزدیكِ", "نزدیكتر", "نزدیك", "نزدِ", "نزديك", "نزد", "نریم", "نره", "نرمی", "نرم", "ندیدی", "ندی", "ندرتا", "ندرت", "نداشتیم", "نداشتید", "نداشتی", "نداشته", "نداشتند", "نداشتم", "نداشت", "نداریم", "ندارید", "نداری", "نداره", "ندارند", "ندارم", "ندارد", "نداد", "نخودی", "نخواهیم", "نخواهید", "نخواهی", "نخواهند", "نخواهم", "نخواهد", "نخستین", "نخستين", "نخست", "نحوه", "نتیجتا", "نبود", "نبش", "نباید", "نبايد", "نباش", "ناید", "ناگهانی", "ناگهان", "ناگزیر", "ناگاه", "ناچار", "ناپذیر", "ناهشیار", "نام", "ناكام", "ناشی", "ناشي", "ناشناخته", "ناسازگارانه", "ناراین", "ناراضی اند", "ناراضی", "نادیده", "ناخوانده", "ناخواسته", "ناتوان", "نااگاهانه", "ناامید", "نا", "ن", "می‌کنیم", "می‌کنند", "می‌کنم", "می‌کند", "می‌فرماید", "می‌شوند", "می‌شود", "می‌شد", "می‌رود", "می‌رسد", "می‌دهد", "می‌داند", "می‌خواهیم", "می‌توانند", "می‌تواند", "می‌توان", "می‌باشند", "می‌باشد", "می‌آید", "میگیره", "میگی", "میگه", "میگن", "میگم", "میگردد", "میکنیم", "میکنید", "میکنی", "میکنه", "میکنند", "میکنن", "میکنم", "میکند", "میکرد", "مینوشند", "میلیون", "میلیارد", "میكنیم", "میكنه", "میكنند", "میكنن", "میكنم", "میكند", "میفرستین", "میفرستی", "میفرستم", "میشوند", "میشود", "میشه", "میشم", "میشد", "میزان", "میروم", "میره", "میرن", "میرم", "میدهیم", "میدهم", "میدهد", "میده", "میدن", "میدم", "میداد", "میخونید", "میخونم", "میخوای", "میخواهند", "میخوانید", "میخوانم", "میخواستن", "میخواد", "میتونه", "میباشد", "میان", "میاد", "می خوانید", "می خوانم", "می", "مگو", "مگه", "مگراینكه", "مگرانكه", "مگر این که", "مگر آن که", "مگر", "مکرراً", "مکرر", "ميليون", "ميليارد", "مي", "مون", "موقعیكه", "موقتا", "موضوع", "مورد", "موخر", "موجودند", "موجب", "موارد", "مواجهند", "مهمتره", "منی", "منو", "منم", "منطقی", "منطقا", "منصفانه", "مند", "منحصربفرد", "منحصرا", "منحصر", "منتهی", "من", "ممیزیهاست", "ممکن", "ملیون", "ملیارد", "ملزم", "مكررا", "مكرر", "مقلوب", "مقصری", "مقصرند", "مقصر", "مقدار", "مقال", "مقابل", "مفیدند", "مغلوب", "مغرضانه", "معمولی", "معمولاً", "معمولا", "معلومه", "معلق", "معذوریم", "معدود", "معتقدیم", "معتقدند", "معتقدم", "مع ذلک", "مع الاسف", "مع", "مطمینا", "مطمنا", "مطمانم", "مطمانا", "مطلقا", "مطلق", "مشكلتر", "مشكل", "مشغولند", "مشغول", "مشخص", "مشترکاً", "مشتركا", "مشت", "مسیولانه", "مسلما", "مسلم", "مستند", "مستمرا", "مستمر", "مستقیما", "مستقیم", "مستقلا", "مستعد", "مستحضرید", "مرسی", "مردم اند", "مردم", "مردانه", "مرتبا", "مرتب", "مراتب", "مرا", "مر", "مذهبی اند", "مذهبی", "مدّتی", "مدتی", "مدتهاست", "مدت", "مدبرانه", "مدبر", "مداوم", "مدام", "مخصوصاً", "مخصوصا", "مختلف", "مختصرا", "مختصر", "مخالفند", "محکم‌تر", "محکم", "محكم", "محض رضای خدا", "محتاطانه", "محتاط", "محتاجند", "مجموعاً", "مجموعا", "مجموع", "مجرمانه", "مجدداً", "مجددا", "مجبورند", "مجانی", "مثلِ", "مثلا", "مثل اینکه", "مثل", "مثابه", "متوسفانه", "متوالی", "متقابلا", "متفكرانه", "متفاوتند", "متعاقبا", "متاسفانه", "متؤسفانه", "مبادا", "مایی", "ماهیتا", "ماهرانه", "ماه", "مانندِ", "مانند", "مان", "مامان مامان گویان", "مامان", "مالامال", "مالا", "ماقبل", "ماشینوار", "ماست", "مارو", "مادامیكه", "مادامی", "مادام", "مات", "ما رو", "ما", "م", "لیکن", "لیكن", "لی", "لکه", "له", "لكه", "لطفاً", "لطفا", "لزوماً", "لزوما", "لذا", "لب", "لاجرم", "لااقل", "لا", "ل", "كیست", "كی", "كَی", "كوركورانه", "كودكانه", "كه", "كنیم", "كنید", "كنی", "كنيم", "كنيد", "كنه", "كننده", "كنند", "كنن", "كنم", "كند", "كنایه", "كنان", "كنارِ", "كنارش", "كنار", "كمی", "كمتره", "كمتر", "كماكان", "كمابیش", "كم", "كلیشه", "كلی", "كلا", "كل", "كشیدن", "كسی", "كسي", "كسانی", "كس", "كزین", "كز", "كردیم", "كردید", "كرده", "كردند", "كردن", "كردم", "كرد", "كرات", "كدامیك", "كدامند", "كدام", "كجاست", "كجا", "كاین", "كان", "كاملتر", "كاملا", "كاشكی", "كاش", "كارند", "كاربرمدارانه", "كارافرینانه", "ك", "قیلا", "قل", "قطعاً", "قطعا", "قضایاست", "قصدِ", "قریب", "قراردادن", "قرار", "قدری", "قدرمسلم", "قدر", "قد", "قبلند", "قبلاً", "قبلا", "قبل", "قانوناً", "قانونا", "قال", "قاعدتاً", "قاعدتا", "قاطعانه", "قاطبه", "قابل", "قاالند", "ق", "فی", "فکر", "فوق", "فوری", "فورا", "فهرستوار", "فناورانه", "فلذا", "فلان", "فكر", "فقط", "فعلاً", "فعلا", "فعالانه", "فرماید", "فردا", "فراوان", "فراتراز", "فراتر", "فر", "فبها", "فاقد", "ف", "غیریكسان", "غیرقانونی", "غیرعمدی", "غیرطبیعی", "غیرتصادفی", "غیرازاین", "غیرازان", "غیراز", "غیر", "غير", "غزالان", "غالبا", "غ", "عیناً", "عینا", "عه", "عنوانِ", "عنوان", "عنقریب", "عن", "عمیقا", "عموماً", "عموما", "عموم", "عملی اند", "عملی", "عملگرایانه", "عملاً", "عملا", "عمل", "عمدی", "عمده", "عمدتاً", "عمدتا", "عمداً", "عمدا", "علیه", "علیرغم", "علی رغم", "علی الظاهر", "علی", "علّتِ", "علناً", "علنا", "علت", "علاوه برآن", "علاوه بر آن", "علاوه بر", "علاوه", "علارغم", "عقِ", "عقبِ", "عقبتر", "عقب", "عضی", "عزیز", "عری", "عرفانی", "عدم", "عجولانه", "عجب", "عج", "عبارتند", "عالی", "عالمانه", "عاقلانه", "عاقبت", "عادلانه", "عاجزانه", "ع", "ظهر", "ظاهراً", "ظاهرا", "ظ", "طی", "طي", "طوری", "طور", "طلبکارانه", "طلبكارانه", "طقِ", "طریق", "طريق", "طرف", "طبیعتا", "طبقِ", "طبعاً", "طبعا", "ط", "ضمناً", "ضمنا", "ضمن", "ضعیفتر", "ضعیف", "ضرورتا", "ضدِّ", "ضدِّ", "ضد", "ض", "صورت", "صندوق هاست", "صندوق", "صمیمانه", "صریحتر", "صریحاً", "صریحا", "صریح", "صرفاً", "صرفا", "صراحتا", "صددرصد", "صدالبته", "صد", "صبح", "صاف", "صادقانه", "ص", "شیک", "شیم", "شیك", "شیرینه", "شیرین", "شی", "شویم", "شوید", "شوی", "شونده", "شوند", "شون", "شوم", "شوقم", "شوراست", "شود", "شو", "شه", "شناسید", "شناسی", "شناسي", "شمایند", "شماست", "شماری", "شما", "شم", "شش نداشته", "شش  نداشته", "شش", "شرط", "شدیم", "شدیداً", "شدیدا", "شدید", "شدی", "شدگان", "شده‌اند", "شده", "شدند", "شدن", "شدم", "شدت", "شد", "شخصاً", "شخصا", "شجاعانه", "شتابزده", "شتابان", "شبهاست", "شبانه", "شب", "شاید", "شايد", "شاهدیم", "شاهدند", "شان", "شاكله", "شادمان", "شادتر", "شاد", "شااالله", "شاأالله", "ش", "سیخ", "سیاه چاله هاست", "سیاه", "سیاری", "سیار", "سی", "سپس", "سویِ", "سوی", "سوي", "سوم", "سهواً", "سهوا", "سه باره", "سه", "سنگین", "سنگدلانه", "سمتِ", "سمت", "سعی", "سعي", "سریِ", "سریعاً", "سریعا", "سریع", "سری", "سرعت", "سراپا", "سرانجام", "سراسر", "سر", "سخته", "سختتر", "سخت", "ست", "سایرین", "سایران", "سایر", "ساکنند", "سالیانه", "سالهاست", "ساله", "سالم‌تر", "سالم", "سالته", "سالانه", "سال", "ساكنند", "ساق", "سازی", "سازگارانه", "سازي", "سازهاست", "سازان", "سادگی", "ساده اند", "ساده", "ساخته‌ام", "ساخته", "ساختن", "سابقا", "سابق", "س", "زین", "زیرچشمی", "زیرِ", "زیرند", "زیركانه", "زیراكه", "زیرا", "زیر", "زیباتر", "زیبا", "زیاده", "زیادتر", "زیاد", "زيرا", "زير", "زياد", "زودی", "زودتر", "زود", "زو", "زهی", "زنند", "زمینه", "زمانی", "زمان", "زشتکارانند", "زشت", "زده", "زدن", "زدم", "زد", "ز", "ریزی", "ریزان", "ریز", "ریاكارانه", "ريزي", "رویِ", "رویم", "رویش", "روی", "روي", "روى", "روهی", "روم", "روشنی", "روشن", "روش", "روزه‌ای", "روزهای", "روزهاي", "روزه م", "روزه ست", "روزه ایم", "روزه", "روزمره", "روزبروز", "روزانه", "روز به روز", "روز", "روبه", "روبروست", "روبرو", "روب", "رواست", "رو", "رهگشاست", "ره", "رندانه", "رنجند", "رفته‌ام", "رفته", "رفتن", "رفتارهاست", "رفت", "رغم", "رشته", "رسیده", "رسید", "رسما", "ردد", "رداری", "رخی", "رخوردار", "رایِ", "رای", "راه", "راستی", "راستا", "راست", "راساس", "راسا", "رارِ", "رادر", "راحتر", "راحت", "راجع به", "راجع", "رابه", "رابسیار", "رااز", "را", "ر", "ذیلا", "ذیل", "ذلک", "ذالك", "ذاشته", "ذاری", "ذاتاً", "ذاتا", "ذ", "دیگه", "دیگری", "دیگرتا", "دیگران", "دیگر", "دیوی", "دیوانه‌ای", "دیوانه", "دیشب", "دیری", "دیروزبه", "دیروز", "دیرم", "دیرت", "دیران", "دیر", "دیده", "دیدم", "دی", "دگرگون", "دگرباره", "دگربار", "دچار", "ديگري", "ديگران", "ديگر", "ديروز", "ديده", "دوهزاری", "دونید", "دونستن", "دون", "دوم", "دوستانه", "دوساله", "دورتر", "دوراز", "دور", "دوتادوتا", "دوتا", "دوباره‌ای", "دوباره", "دو روزه", "دو", "دهیم", "دهید", "دهی", "دهند", "دهم", "دهد", "ده", "دنبالِ", "دنبال", "دنالِ", "دم", "دلشاد", "دلخوش", "دلخواه", "دقیقا", "دقیق", "دفعه", "دشوارتر", "دشوار", "دشمنیم", "دسته دسته", "دسته", "درین", "دریغا", "دریغ", "درپی", "درون", "درواقع", "درهرصورت", "درهرحال", "درهر", "درنهایت", "درنتیجه", "درمیان", "درمورد", "درمقابل", "درمجموع", "دركنار", "دركل", "درعین حال", "درعین", "درطی", "درصورتیكه", "درصورتی که", "درصورتی", "درشتی", "درشت", "درسته", "درست و حسابی", "درست", "درراستای", "دردكشان", "درحالیکه", "درحالیكه", "درحالی که", "درحالی", "درحال", "درثانی", "درتخت", "دربه", "دربرابر", "دربر", "دربدر", "درباره", "درباب", "دراین میان", "دراین", "دران", "درازای", "درازا", "دراره", "دراثر", "در کنار", "در کل", "در واقع", "در نهایت", "در مجموع", "در ثانی", "در بارهٌ", "در باره", "در آینده", "در", "دخترانه", "دایما", "دایم", "داوطلبانه", "دانند", "دانست", "دامم", "داشتیم", "داشتید", "داشتی", "داشته", "داشتند", "داشتن", "داشتم", "داشت", "داریم", "دارید", "داری", "داريم", "داره", "دارند", "دارن", "دارم", "دارد", "دارای", "داراست", "دارا", "دار", "دادیم", "دادید", "دادی", "داده", "دادند", "دادن", "دادم", "داد", "داخل", "دااما", "داام", "دا", "د", "خیلی", "خیره", "خیر", "خیاه", "خویشتنم", "خویشتن", "خویش", "خوگیرانه", "خويش", "خوشحال", "خوشبینانه", "خوشبختانه", "خوش", "خوردن", "خودی", "خودنمایانه", "خودمو", "خودمختارانه", "خودمان", "خودم", "خودشون", "خودشان", "خودش", "خودرا", "خودتو", "خودتان", "خودت", "خودبه خودی", "خودبه", "خوداگاهانه", "خوداند", "خود به خود", "خود", "خوتهد", "خوبی", "خوبست", "خوبتر", "خوب", "خواهیمكرد", "خواهیم", "خواهید", "خواهی", "خواهيم", "خواهند", "خواهم", "خواهش", "خواهد", "خواه", "خواستیم", "خواستید", "خواستی", "خواسته", "خواستند", "خواستن", "خواستم", "خواست", "خو", "خلاقانه", "خلاصه", "خصوصاً", "خصوصا", "خصمانه", "خشمگین", "خسته‌ای", "خسته", "خردمندانه", "خدمات", "خداگونه", "خداست", "خداروشکر", "خداحافظ", "خب", "خامسا", "خام", "خالی", "خالصانه", "خالص", "خاست", "خارجِ", "خارج", "خ", "حکماً", "حول", "حكیمانه", "حكما", "حقیقتا", "حقیرانه", "حق", "حضرتعالی", "حسب", "حسابی", "حسابگرانه", "حرف", "حدودِ", "حدودا", "حدود", "حداکثر", "حداكثر", "حداقل", "حتی", "حتي", "حتماً", "حتما", "حاکیست", "حالی", "حالكه", "حالا", "حال", "حاكیست", "حاضرم", "حاضر", "حاشیه‌ای", "حاشیه", "حاشاوكلا", "حاشا", "ح", "جوری", "جور", "جهت", "جنس اند", "جنس", "جناح", "جنابعالی", "جمله", "جمعی", "جمعا", "جمع اند", "جمع", "جلویِ", "جلویری", "جلوی", "جلوگیری", "جلوگيري", "جلوتر", "جلو", "جسورانه", "جزجز", "جز", "جریان", "جريان", "جرمزاست", "جدیدا", "جدید", "جدی", "جديد", "جداگانه", "جداً", "جداازهم", "جدااز", "جدا", "جبرگرایانه", "جایی", "جای", "جايي", "جاي", "جا", "ج", "ثانیاً", "ثانیا", "ثانی", "ثالثاً", "ثالثا", "ث", "تی", "تک تک", "تک", "تویی", "تویِ", "توی", "تووما", "تونست", "تون", "تولِ", "توسط", "توانیم", "توانید", "توانی", "توانند", "توانم", "توانستیم", "توانستی", "توانسته", "توانستند", "توانستن", "توانستم", "توانست", "تواند", "توان", "توؤماً", "تو", "ته", "تنگاتنگ", "تنهایی", "تنها", "تند تند", "تند", "تمامی", "تمامي", "تمامشان", "تماما", "تمام قد", "تمام", "تلویحاً", "تلویحا", "تك", "تقریباً", "تقریبا", "تفننی", "تفاوتند", "تفاوت", "تعمدا", "تعدادی", "تعداد", "تصریحاً", "تصریحا", "ترین", "تریلیون", "تریلیارد", "تری", "ترين", "ترند", "تردید", "ترجیحا", "ترتیب", "تر براساس", "تر  براساس", "تر", "تدریجی", "تدریجا", "تدریج", "تحقیق", "تحریم هاست", "تحریم", "تحت", "تاکنون", "تاوقتیكه", "تاوقتی", "تان", "تاكنون", "تازگی", "تازه", "تاحدی", "تاحدودی", "تاجاییكه", "تابه", "تااینكه", "تااینجا", "تاانكه", "تاانجاكه", "تاانجا", "تا", "ت", "بیگمان", "بیهوده", "بینمان", "بینابین", "بین", "بیفته", "بیشتری", "بیشتر", "بیش", "بیست", "بیرونِ", "بیرون", "بیاییم", "بیایید", "بیایی", "بیایند", "بیایم", "بیاید", "بیاوریم", "بیاورید", "بیاوری", "بیاورند", "بیاورم", "بیاورد", "بیاور", "بیام", "بیاریم", "بیاد", "بیابیم", "بیابید", "بیابی", "بیابند", "بیابم", "بیابد", "بیاب", "بیا", "بی هدف", "بی نیازمندانه", "بی تفاوتند", "بی تردید", "بی اطلاعند", "بی آنکه", "بی", "بگین", "بگیریم", "بگیرید", "بگیری", "بگیرند", "بگیرم", "بگیرد", "بگیر", "بگی", "بگوییم", "بگویید", "بگویی", "بگویند", "بگویم", "بگوید", "بگو", "بگه", "بگم", "بگرمی", "بگذاریم", "بکنیم", "بکنید", "بکنی", "بکنند", "بکنم", "بکند", "بکن", "بکار", "بپا", "بين", "بيشتري", "بيشتر", "بيش", "بيست", "بي", "بویژه", "بوضوح", "بودیم", "بودید", "بودی", "بوده", "بودند", "بودن", "بودم", "بود", "بوجود", "بهیچ", "بهم", "بهشون", "بهش", "بهرحال", "بهتون", "بهترین", "بهترين", "بهتر", "بهت", "به گرمی", "به کرات", "به ویژه", "به وضوح", "به هیچ وجه", "به هرحال", "به ناچار", "به مراتب", "به قدری", "به علاوه", "به طوری که", "به طور کلی", "به طور", "به شدت", "به شان", "به سرعت", "به سادگی", "به زودی", "به روشنی", "به رغم", "به راستی", "به دلخواه", "به درشتی", "به خوبی", "به جز", "به جای", "به تمامی", "به تدریج", "به تازگی", "به آسانی", "به", "بنی", "بندی", "بندي", "بندرت", "بناچار", "بنابه", "بنابرین", "بنابراین", "بنابراين", "بموقع", "بموجب", "بمراتب", "بماند", "بلی", "بلکه", "بله", "بلكه", "بلافاصله", "بلادرنگ", "بكرات", "بكار", "بقدری", "بفهمی نفهمی", "بفهمی", "بعید", "بعلاوه", "بعضی‌ها", "بعضیهایشان", "بعضیهاشان", "بعضیها", "بعضیشان", "بعضی شان", "بعضی", "بعضي", "بعضا", "بعری", "بعدی", "بعدها", "بعداً", "بعدازظهر", "بعدازاین", "بعدازان", "بعداز", "بعدا", "بعد از این که", "بعد", "بطوریكه", "بطوری که", "بطوری", "بطوركلی", "بطور", "بطبع", "بصورت", "بشین", "بشیم", "بشید", "بشی", "بشوند", "بشه", "بشن", "بشم", "بشدت", "بشان", "بسیاری", "بسیار", "بسی", "بسياري", "بسيار", "بسوی", "بسهولت", "بسرعت", "بسختی", "بسته", "بسادگی", "بسا", "بس", "بزودی", "بزنید", "بزعم", "بزرگ", "بریم", "بریزید", "برید", "بری", "برپا", "بروشنی", "بروز", "برو", "بره", "برنمی", "برنامه سازهاست", "برنامه", "برن", "برمی", "برم", "برغم", "برعکس", "برعكس", "برروی", "بردن", "برداشتن", "برداری", "برداري", "برخی", "برخي", "برخوردارند", "برخوردار", "برخلاف", "برحسب", "برایِ", "برایمان", "برایم", "برایشان", "برایش", "برایت", "برای", "براي", "برانهاست", "برانند", "بران", "براستی", "براساس", "براحتی", "براثر", "برابرِ", "برابر", "برا", "برآنند", "برآن", "بر", "بدینسان", "بدینجا", "بدین ترتیب", "بدین", "بدیم", "بدید", "بدون", "بدهید", "بده", "بدم", "بدلخواه", "بدرشتی", "بدرستی", "بدخواهانه", "بدجور", "بدبینانه", "بدانید", "بدانها", "بدانجا", "بدان", "بد جور", "بد", "بخوبی", "بخواهیم", "بخواهید", "بخواهی", "بخواهند", "بخواهم", "بخواهد", "بخواه", "بخصوص", "بخشی", "بخشه", "بخش", "بخردانه", "بخاطراینكه", "بخاطر", "بجز", "بجای", "بجا", "بتوانیم", "بتوانی", "بتواند", "بتوان", "بتمامی", "بتدریج", "بتازگی", "ببینید", "ببینم", "بایستی", "باید", "بايد", "باورند", "باوجودیكه", "باوجودی که", "باوجودی", "باوجوداینكه", "باوجودانكه", "باوجود", "باهم", "بالنسبه", "بالنتیجه", "بالله", "بالقوه", "بالعکس", "بالعكس", "بالطبع", "بالضرور", "بالایِ", "بالای", "بالاست", "بالاخص", "بالاخره", "بالاتر", "بالا", "باعلاقه", "باشیم", "باشید", "باشی", "باشيم", "باشه", "باشند", "باشم", "باشد", "باش", "باستثنای", "بازیگوشانه", "بازی کنان", "بازی", "بازهم", "بازم", "بازاندیشانه", "باز هم", "باز", "بارهٌ", "بارهاوبارها", "بارها", "باره", "بارة", "بار", "باد", "باتوجه", "بااینكه", "بااین وجود", "بااین حال", "بااین", "باانكه", "بااطمینان", "با", "ب", "این‌ها", "اینگونه", "اینکه", "اینک", "اینچنین", "اینو", "اینهمه", "اینهاست", "اینها", "اینه", "اینم", "اینكه", "اینك", "اینقدر", "اینطوری", "اینطور", "اینست", "اینرو", "ایند", "اینجوری", "اینجور", "اینجاست", "اینجا", "اینان", "اینا", "این گونه", "این قدر", "این دفعه", "این جوری", "این", "ایم", "ایشان", "اید", "ایا", "ای", "اگه", "اگرچه", "اگرنه", "اگر چه", "اگر", "اگاهانه", "اکنون", "اکثریت", "اکثراً", "اکثرا", "اکثر", "اينكه", "اين", "ايم", "ايشان", "اي", "اویی", "اوه", "اونهمه", "اونجوری که", "اونجوری", "اونجور", "اونجا", "اونایی", "اونا", "اون", "اومده", "اومد", "اولین", "اولش", "اولاً", "اولا", "اول", "اوست", "اوردن", "اورد", "اور", "او", "اهای", "اهان", "اه", "انگونه", "انگه", "انگاه", "انگار", "انکه", "انچه", "انچنان", "انوقت", "انهاست", "انها", "انم", "انكه", "انكس", "انقدر", "انطور", "انصافا", "انشالا", "انشاالله", "انرا", "اندکی", "اندكی", "اندك", "اند", "انجام", "انتها", "انانی", "انان", "ان شاأالله", "ان", "امیدواریم", "امیدوارند", "امیدوارم", "امورات", "امور", "امشب", "امسال", "امروزه", "امروز", "امرانه", "امان", "اما", "ام", "الی", "الهی", "المقدور", "الظاهر", "الزاما", "التّه", "الته", "البتّه", "البته", "الایِ", "الان", "الاسف", "الا", "اكنون", "اكثریت", "اكثرا", "اكثر", "اكتسابا", "اقلیت", "اقلا", "اقل", "افقی", "افسوس", "افزود", "افتادن", "افتاد", "اغلب", "اعلام", "اطلاعند", "اصولاً", "اصولا", "اصلاً", "اصلا", "اصطلاحا", "اصطلاجا", "اشیم", "اشکارا", "اشند", "اشنایند", "اشكارتر", "اشكارا", "اشكار", "اشفته", "اشد", "اشتباها", "اشان", "اش", "اسلامی اند", "استفاده", "استفاد", "است", "اسانی", "اسانتر", "اسان", "اساساً", "اساسا", "اساس", "اس", "ازیك", "ازو", "ازنظر", "ازلحاظ", "ازقبیل", "ازش", "ازسر", "ازروی", "ازجمله", "ازبه", "ازاینرو", "ازاین رو", "ازاین", "ازانجاكه", "ازانجا", "ازان", "ازادانه", "از جمله", "از بس که", "از آن پس", "از", "اری", "اره", "ارنه", "ارایه", "ارام", "اراسته", "اخیراً", "اخیرا", "اخیر", "اخه", "اخرها", "اخر", "اختصارا", "اخ", "احیاناً", "احیانا", "احتمالا", "احتراما", "اجراست", "اثرِ", "اثر", "اتفاقا", "ات", "ابلهانه", "ابدا", "ا", "آیند", "آید", "آیا", "آی", "آيد", "آوه", "آورده", "آوردن", "آورد", "آور", "آهای", "آهان", "آن‌ها", "آنگاه", "آنکه", "آنچه", "آنچنان که", "آنچنان", "آنهاست", "آنها", "آنكه", "آنقدر", "آنطور", "آنرا", "آنجا", "آنانی", "آنان", "آن گاه", "آن", "آمرانه", "آمده", "آمدن", "آمد", "آقایان", "آقای", "آقا", "آشکارا", "آشنایند", "آسیب پذیرند", "آسیب", "آسانی", "آسان", "آزادانه", "آری", "آره", "آرام آرام", "آرام", "آدمهاست", "آخه", "آخرها", "آخر", "آخ", "آباد"]
```

## Remove stopwords
```python
def remove_stopword(inputString):
    stop_words = set(stoplist) 
    word_tokens = word_tokenize(inputString) 
    filtered_sentence = [w for w in word_tokens if not w in stop_words] 
    filtered_sentence = [] 
    for w in word_tokens: 
        if w not in stop_words: 
            filtered_sentence.append(w)

    return " ".join(filtered_sentence)
```

## Create wordcloud
```python
file = "chat.txt"
data = os.path.dirname(__file__) if "__file__" in locals() else os.getcwd()
file_input = codecs.open(os.path.join(data, file), 'r', 'utf-8')
chat_text = arabic_reshaper.reshape(remove_emoji(file_input.read()))
chat_text = chat_text.replace("\n", " ")
chat_text = remove_garbage(chat_text)
chat_text = remove_stopword(chat_text)
chat_text = get_display(chat_text)
wc = WordCloud(background_color="white",font_path='vazir.ttf', width=2000, height=1000)
wc.generate(chat_text)
print(file)
plt.figure(figsize=(20,20))
plt.axis("off")
plt.imshow(wc)
plt.savefig("chat.png")
plt.show()
```
