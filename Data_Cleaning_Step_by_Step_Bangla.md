# Data Cleaning: ধাপে ধাপে বাংলা পাঠ ও কাজের checklist

চারটি ভিডিওর ব্যবহারযোগ্য শিক্ষা, তোমার মূল Power Query গাইড এবং আমাদের অনুশীলন মিলিয়ে তৈরি। মূল কাজের পরিবেশ Excel ও Power Query; তথ্যের অর্থ বুঝে সিদ্ধান্ত নেওয়ার যুক্তি SQL ও Python-এও কাজে লাগে।

**Cleaning-এর লক্ষ্য: তথ্যের অর্থ ঠিক রেখে নির্দিষ্ট হিসাবের জন্য data-কে ব্যবহারযোগ্য করা।**

এই ১২টি ধাপ তোমার সাধারণ কাজের পথ। প্রতিটি ধাপে আগে দেখবে কাজটি দরকার কি না। যেখানে পরিবর্তন দরকার নেই, সেখানে পরীক্ষা করে এগিয়ে যাবে। কোনো পরের ধাপ নতুন সমস্যা তৈরি বা প্রকাশ করলে আগের সংশ্লিষ্ট ধাপে ফিরে আসবে।

**দুটি বিষয় আলাদা:** “আগে–পরে করা যায়” মানে কাজের ক্রম বদলানো যায়। “বাদ দেওয়া যায়” মানে ওই dataset-এ কাজটিই প্রয়োজন নেই।

| ধাপ | কাজ | কখন দরকার |
|---|---|---|
| ১ | লক্ষ্য, row-এর অর্থ ও key বোঝা | প্রতিটি নতুন কাজের শুরুতে |
| ২ | Raw data রাখা ও নিরাপদ import | পরিবর্তন করার আগে |
| ৩ | সমস্যা খোঁজা ও শুরুর অবস্থা লেখা | পরিবর্তন বাছাই করার আগে |
| ৪ | দরকারি context উদ্ধার ও table-এর গঠন ঠিক করা | Layout-এ সমস্যা থাকলে |
| ৫ | মিশে থাকা তথ্য Split/Extract করা | আলাদা field প্রয়োজন হলে |
| ৬ | Text, key ও category standardise করা | অর্থহীন পার্থক্য থাকলে |
| ৭ | Missing value, Number ও Date সামলানো | Missing/ভুল type/অস্পষ্ট format থাকলে |
| ৮ | Duplicate, revision ও বৈধ detail আলাদা করা | Repeated record বা version থাকলে |
| ৯ | Append, Merge, Pivot/Unpivot | একাধিক table বা ভিন্ন layout প্রয়োজন হলে |
| ১০ | প্রয়োজনীয় হিসাব ও status তৈরি | নতুন measure বা flag দরকার হলে |
| ১১ | ফল যাচাই করা | Output বিশ্বাস করার আগে |
| ১২ | সিদ্ধান্ত লিখে রাখা, load ও refresh পরীক্ষা | Handover-এ; পুনরাবৃত্ত কাজ হলে refresh-ও |

## ধাপ ১ — কী হিসাব চাই, এক row কী বোঝায়, key কী—বোঝো

কাজ শুরু করার আগে চারটি প্রশ্নের উত্তর দাও:

- কী প্রশ্নের উত্তর দিতে হবে? কোন সময়কাল ও কোন records এর আওতায়?
- এক row কী বোঝায়? এটিই **Grain**।
- কোন field বা fields মিলে record আলাদা করে? এটিই **Key**।
- কোন columns ওই হিসাবের জন্য দরকার, এবং সেগুলোর অর্থ কী?

Booking table-এ এক row এক booking হতে পারে। Stop table-এ এক row এক stop; সেখানে `Booking_ID + Stop_No` লাগতে পারে।

**Measure** হলো হিসাবের মান, যেমন pallets বা cost। **Dimension** হলো কোন দিক দিয়ে ভাগ করে দেখবে, যেমন Site বা Carrier। দেখতে সংখ্যা হলেও Booking_ID একটি পরিচয়; যোগ করার quantity নয়।

গুরুত্বপূর্ণ column হিসাব অনুযায়ী বদলায়। Booking count-এর জন্য Left অপরিহার্য নয়; time on site-এর জন্য Arrived ও Left দুটিই দরকার। “Date” নামে একটি column দেখেই সেটি booking date না arrival date অনুমান কোরো না।

**ক্রম:** Meaning, grain ও key বুঝে তারপর record বাদ দেওয়া, category এক করা বা totals তৈরি করা নিরাপদ হয়।

**বাদ দেওয়া যাবে?** এই বোঝাপড়া প্রয়োজন। আগে থেকে নির্ভরযোগ্য সংজ্ঞা থাকলে সেটি ব্যবহার করবে; নতুন করে সব লিখতে হবে না।

## ধাপ ২ — Raw data অক্ষত রাখো এবং import-এ তথ্য রক্ষা করো

Raw হলো পাওয়া মূল data। আলাদা working copy বা transformation query-তে কাজ করো। Source file, প্রয়োজনীয় source-row reference এবং পরিবর্তনের কারণ রাখো। Source row number সবসময় business key নয়।

**ID `001` হলে:** Import-এর প্রথম conversion-এই Text রাখো। Auto-generated type step যদি সেটিকে `1` করে, সেই আগের step ঠিক করে raw source থেকে আবার পড়ো। শুধু শেষে Text করলে `"1"` পাওয়া যাবে।

নিয়ম অনুযায়ী ID অবশ্যই ছয় অঙ্কের এবং বাকি মান বৈধ হলে `32`-কে `"000032"` হিসেবে pad করা যায়। এটি জানা নিয়ম প্রয়োগ করা; অজানা original zero-count আন্দাজ করা নয়। Blank, অতিরিক্ত লম্বা বা invalid ID আগে flag করবে।

Excel-এ helper column দিয়ে কাজ করলে source ও cleaned result পাশাপাশি যাচাই করতে পারো। পরে স্থায়ী ফল চাইলে Paste Values ব্যবহার করা যায়। Raw copy আলাদা থাকবে।

**ক্রম:** গুরুত্বপূর্ণ leading zero রক্ষা → lossy conversion হওয়ার আগেই। Raw রাখা → মূল তথ্য বদলানোর আগে।

**বাদ দেওয়া যাবে?** মূল তথ্য ফিরে পাওয়ার ব্যবস্থা প্রয়োজন। Copy বা version আগে থেকেই থাকলে সেটি ব্যবহার করো।

## ধাপ ৩ — আগে সমস্যা খুঁজে বের করো; baseline লেখো

Baseline হলো পরিবর্তনের আগের অবস্থা, যার সঙ্গে পরে ফল মিলিয়ে দেখবে।

- Source rows কত? প্রকৃত data rows কত? Heading/subtotal আলাদা কি?
- Column names ও types কী?
- Missing, error, repeated key এবং অচেনা category কোথায়?
- Quantity/date-এর অস্বাভাবিক মান ও প্রয়োজনীয় totals কী?
- কোনো column-এর রং, heading বা file name কি দরকারি অর্থ বহন করছে?

**Excel:** Filter দিয়ে distinct values দেখো; `ISNUMBER`, `ISTEXT`, `LEN` ও প্রয়োজনীয় counts ব্যবহার করো। AutoFit দরকার হলে করো, যাতে লেখা দেখা যায়।

**Power Query:** Column quality, distribution ও profile দেখো। Profiling সাধারণত প্রথম ১,০০০ row-এর ওপর হয়; পুরো dataset যাচাই করতে entire dataset নির্বাচন করো। [১]

`ISNUMBER=TRUE` শুধু numeric type বোঝায়; সঠিক date বা business value প্রমাণ করে না। `FALSE` মানেই Text নয়। `LEN=6` ছয়টি character বোঝায়; ID বৈধ বা unique কি না তা আলাদা পরীক্ষা। [২]

**ক্রম:** সমস্যা শনাক্ত → সেই সমস্যার জন্য operation বাছাই।

**বাদ দেওয়া যাবে?** অন্তত প্রয়োজনীয় fields ও পুরো data-এর যথাযথ পরীক্ষা দরকার। সব rows হাতে পড়ে দেখার প্রয়োজন নেই।

## ধাপ ৪ — Context উদ্ধার করে table-এর গঠন ঠিক করো

Context হলো record বুঝতে প্রয়োজনীয় বাড়তি তথ্য—যেমন Site শুধু ওপরের heading-এ লেখা আছে।

| Site | Booking_ID |
|---|---|
| Leicester | null |
| null | 001 |
| null | 002 |

যদি নিশ্চিত হও প্রথম row heading এবং নিচের দুই booking ওই Site-এর, তাহলে **Site Fill Down → heading বাদ**। আগে heading মুছলে ওই ধাপে Site-এর উৎস থাকবে না।

Fill Down-এর আগে জানতে হবে: কোন group-এর মধ্যে মানটি প্রযোজ্য, row order ঠিক কি না, কোথায় group শেষ। Group ভেঙে যায় এমন Sort বা Filter আগে কোরো না। Power Query Fill Down `null` পূরণ করে; empty text বা spaces হলে অর্থ বুঝে আগে null mapping লাগতে পারে। [৩]

এখানেই প্রয়োজনমতো ঠিক করবে:

- প্রকৃত header শনাক্ত করা; প্রতিটি column-এর স্পষ্ট, আলাদা নাম দেওয়া।
- Title, repeated header, subtotal ও footer-কে প্রকৃত records থেকে আলাদা করা।
- নিশ্চিত অপ্রয়োজনীয় spacer বাদ দেওয়া। Blank row group boundary হলে আগে তার অর্থ সংরক্ষণ করা।
- Merged layout-এর তথ্য প্রয়োজন হলে সঠিক records-এ বহন করা।

**পুরো blank row আর একটি blank cell এক নয়।** Excel-এর Go To Special → Blanks blank cells নির্বাচন করে। সেখান থেকে পুরো rows মুছলে দরকারি partial record-ও হারাতে পারে। [৪]

পুরো blank spacer আগে চিহ্নিত বা সরিয়ে নাও: Fill Down করার পরে সেখানে Site বসলে সেটি আর পুরো blank থাকবে না।

**বাদ দেওয়া যাবে?** Header, context ও layout ইতিমধ্যে ঠিক থাকলে এই পরিবর্তনগুলো লাগবে না। ব্যবহারযোগ্য layout হলে Excel Table বানানো যায়; একেবারে শেষ পর্যন্ত অপেক্ষা বাধ্যতামূলক নয়।

## ধাপ ৫ — এক column-এ মিশে থাকা তথ্য প্রয়োজন হলে আলাদা করো

ধরো `Trip = "A01 - Paris"`। আলাদা code দিয়ে matching বা grouping দরকার হলে Trip_Code ও Trip_Name বানাও।

- Separator দিয়ে আলাদা হলে **Split by delimiter**।
- অবস্থান সত্যিই নির্দিষ্ট হলে **Fixed width** বা `LEFT/RIGHT/MID`।
- Length বা separator-এর অবস্থান বদলালে সেই নিয়ম বুঝে `LEN/SEARCH` বা উপযুক্ত extraction ব্যবহার করো।

Code-এর পর প্রথম hyphen-এ split দরকার, নাকি সব hyphen-এ—তা আগে ঠিক করো। নামের নিজের মধ্যেও hyphen থাকতে পারে। Text to Columns-এর destination খালি রাখো এবং ID-এর type রক্ষা করো।

Split-এর পরে নতুন অংশের বাইরের space দেখো: `"A01 "` ও `" Paris"` হলে Trim দরকার। **পুরো string আগে Trim করলেও ভেতরের hyphen-এর পাশের space থেকে যেতে পারে।**

Flash Fill ব্যবহার করলে ব্যতিক্রমী নাম/rows-এ ফল যাচাই করো। প্রথম space ধরে split করলে full name-এর বদলে শুধু প্রথম অংশ পাওয়া যেতে পারে।

**ক্রম:** Source-এর গঠন বোঝা → Split/Extract → নতুন অংশ যাচাই ও প্রয়োজনীয় Trim।

**বাদ দেওয়া যাবে?** দরকারি fields ইতিমধ্যে আলাদা থাকলে, বা আলাদা করার কোনো analytical প্রয়োজন না থাকলে।

## ধাপ ৬ — Text, key ও category standardise করো

Standardise মানে একই অর্থের মানকে অনুমোদিত একই রূপে লেখা।

| কাজ | ছোট উদাহরণ | শর্ত |
|---|---|---|
| Trim | `" 001 "` → `"001"` | বাইরের space অর্থহীন |
| Case এক করা | `"dhl"` → `"DHL"` | Case ব্যবসার পরিচয় বদলায় না |
| Clean/নির্দিষ্ট character বদলানো | Control character সামলানো | দরকারি word boundary রক্ষা করতে হবে |
| Alias mapping | দুইটি যাচাইকৃত বানান → অনুমোদিত নাম | একই entity বলে নিশ্চিত |

`DHLL`-কে শুধু দেখতে কাছাকাছি বলে `DHL` বানাবে না। অচেনা category-এর raw value রেখে “যাচাই প্রয়োজন” flag দাও। Spell Check দিয়ে ব্যবসার পরিচয় নিশ্চিত করা যায় না।

Excel `TRIM` সাধারণ internal repeated spaces-ও একটিতে নামায়; Power Query `Text.Trim` মূলত শুরু/শেষের whitespace সরায়। Excel TRIM একা nonbreaking space সরায় না। Clean-ও সব অদৃশ্য character-এর একমাত্র সমাধান নয়; line break মুছলে দুটি শব্দ জোড়া লেগে যেতে পারে। [৫]

Find/Replace নির্দিষ্ট range-এ করো; দরকারমতো whole-cell matching ব্যবহার করো। Literal `*` খুঁজতে `~*`; `*` নিজে wildcard। পূর্বের format criteria সক্রিয় আছে কি না দেখো। [৬]

**ক্রম:** যে মান দিয়ে equality matching হবে, সেই মানের প্রয়োজনীয় standardisation আগে বা matching-এর মধ্যেই করতে হবে।

**বাদ দেওয়া যাবে?** প্রয়োজনীয় text/key/category ইতিমধ্যে সঠিক ও consistent হলে।

## ধাপ ৭ — Missing values-এর অর্থ বুঝে Number ও Date ঠিক করো

প্রথমে blank বা marker-এর অর্থ বোঝো: অজানা, প্রযোজ্য নয়, এখনও ঘটেনি, নাকি আগের heading-এর তথ্য উত্তরাধিকারসূত্রে প্রযোজ্য?

এই column-এ `N/A` অজানা quantity বোঝায় বলে নিশ্চিত হলে:

`" N/A "` → Trim → `null`; সঙ্গে প্রয়োজনীয় Status।

অন্য booking-এর Actual বা Left এখানে বসাবে না। অজানা মানে `0` নয়। Number/Date column-এ `Unknown` Text ঢোকানোর বদলে null রেখে আলাদা status রাখো। Marker-এর raw value মুছে ফেলবে না।

তারপর type পরীক্ষা ও প্রয়োজনে conversion করো:

| তথ্য | উপযুক্ত ধরন |
|---|---|
| Booking_ID / code | পরিচয়ের নিয়ম অনুযায়ী Text |
| পূর্ণ pallets | Whole Number |
| Cost | প্রয়োজনীয় precision-সহ Number; currency পরিচয়ও রাখতে হবে |
| কেবল তারিখ | Date |
| Arrival/departure timestamp | DateTime; প্রয়োজন হলে timezone-ও বুঝতে হবে |

Excel-এ numeric text-এর জন্য `VALUE` বা উপযুক্ত conversion; Power Query-তে সঠিক Data Type এবং প্রয়োজন হলে locale ব্যবহার করো। Numbers-এর decimal/thousands separator, units ও sign-এর অর্থ বুঝে নাও। CR/DR কোন sign বোঝায় তা report-specific।

**Date-এর নিয়ম:** `21/09/2026` যদি DMY এবং `2026-09-21` যদি YMD বলে জানা থাকে, দুটিকে সরাসরি একই Date value-তে parse করা যায়। আগে একই Text display বানানো বাধ্যতামূলক নয়। `04/05/2026`-এর format অজানা হলে ৪ মে নাকি ৫ এপ্রিল অনুমান করবে না; raw রাখবে এবং flag করবে। [৭]

Excel-এ Number/Date/Currency format নির্বাচন প্রদর্শন বদলায়। সেটি সব Text number/date convert করে না। Numeric `10.75`-কে `£10.75` দেখালেও মূল মান numeric থাকে। `TEXT()` দিয়ে তৈরি ফল আবার Text হয়। [৮]

**ক্রম:** প্রয়োজনীয় marker handling ও source format বোঝা → conversion → type ও অর্থ পুনরায় পরীক্ষা। ID-এর protection ধাপ ২-তেই হবে; সব type শেষের জন্য অপেক্ষা করবে না।

Conversion error হলে আগে raw value দেখে কারণ বোঝো। শুধু Remove Errors দিয়ে row মুছলে সেই row-এর অন্য ব্যবহারযোগ্য তথ্যও হারাতে পারে। প্রয়োজন হলে invalid field আলাদা flag করো এবং কোন metric-এ সেটি ব্যবহার করা যাবে তা ঠিক করো।

**বাদ দেওয়া যাবে?** Missing handling দরকার না হলে এবং types/অর্থ ঠিক থাকলে আবার conversion লাগবে না।

## ধাপ ৮ — Duplicate, revision ও বৈধ একাধিক record আলাদা করো

| একই ID আবার আছে কেন? | কী করবে |
|---|---|
| নিশ্চিত repeated export copy | নির্ধারিত নিয়মে একটি রাখবে |
| আগের record-এর revision | বর্তমান report-এর জন্য কার্যকর version; history দরকার হলে versions সংরক্ষণ |
| আলাদা stop/item/event | বৈধ detail রাখবে; উপযুক্ত composite key ব্যবহার |

শুধু সব visible cells একই হলেই business duplicate প্রমাণ হয় না; প্রয়োজনীয় পরিচয়ের field export-এ বাদ পড়েও থাকতে পারে।

Duplicate comparison-এর columns সচেতনভাবে বেছে নাও। SourceRow আলাদা বলে একই business record-এর repeated copy আলাদা মনে হতে পারে। আবার শুধু Booking_ID তুলনা করলে বৈধ stops হারাতে পারে। Raw/provenance আলাদা রেখে business comparison-এর fields ঠিক করো।

Power Query তুলনায় নির্বাচিত columns ও case বিবেচনা করে। Carrier তুলনায় থাকলে প্রয়োজনীয় case/space পরিষ্কার করবে; শুধু Booking_ID তুলনায় থাকলে Carrier পরিবর্তন ওই তুলনার পূর্বশর্ত নয়। কোন copy থাকবে তা গুরুত্বপূর্ণ হলে স্পষ্ট নির্বাচন-নিয়ম দাও—শুধু Sort + Remove Duplicates করলেই পছন্দের প্রথম row থাকবে ধরে নিও না। [৯]

**ক্রম:** Grain ও retention rule বোঝা → তুলনার প্রয়োজনীয় মান প্রস্তুত → duplicate/revision handling। Source আগেই নির্ভরযোগ্যভাবে exact copy চিহ্নিত করলে আরও আগেও তা সরানো সম্ভব।

**বাদ দেওয়া যাবে?** Repetition বা version সমস্যা না থাকলে পরিবর্তন লাগবে না; key পরীক্ষা প্রয়োজন অনুযায়ী করবে।

## ধাপ ৯ — প্রয়োজন হলে tables যোগ করো বা layout বদলাও

| কাজ | অর্থ | কী যাচাই করবে |
|---|---|---|
| Append | এক ধরনের rows একটির নিচে আরেকটি | Column meaning, schema, units, overlapping records |
| Merge | Key মিলিয়ে অন্য table-এর তথ্য আনা | Key types/normalisation, lookup uniqueness, unmatched records |
| Unpivot | Measure columns-কে attribute/value rows-এ নামানো | নতুন grain/key ও missing-value coverage |
| Pivot | Row-এর category/measure-কে পাশাপাশি columns-এ তোলা | একাধিক value থাকলে কোন aggregation বৈধ |

এক booking-এর Plan ও Actual Unpivot করলে এক row = এক booking-এর এক measure। তখন শুধু Booking_ID দিয়ে duplicate মুছলে বৈধ row হারাবে। Power Query Unpivot-এ null pair বাদ যেতে পারে; অনুপস্থিত row-কে zero বলবে না। [১০]

Merge expand করার পরে rows বাড়লে দেখো এটি প্রত্যাশিত one-to-many সম্পর্ক, নাকি duplicate lookup key-এর সমস্যা। Intended grain না বুঝে আবার duplicate সরিয়ে সমস্যাটি ঢাকবে না।

Folder combine-এ সঠিক files/schema নির্বাচন করো; temporary, output বা অনিচ্ছাকৃত files বাদ দাও। প্রয়োজনীয় source filename রাখো। একই compatible file rules পরের refresh-এও প্রযোজ্য হতে হবে। [১১]

**ক্রম:** Matching-এর প্রয়োজনীয় key প্রস্তুত → Merge। Reshape-এর পরে grain ও key আবার যাচাই। Layout ব্যবহারযোগ্য করার জন্য reshape আগে দরকার হলে ধাপ ৪-এই করতে পারো; তৈরি হওয়া columns পুনরায় inspect করবে।

**বাদ দেওয়া যাবে?** একটি table-এই দরকারি তথ্য ও উপযুক্ত layout থাকলে। Wide table নিজে থেকে ভুল নয়।

## ধাপ ১০ — প্রয়োজনীয় calculation ও status তৈরি করো

যেমন `Time on site = Left − Arrived`, অথবা দুই quantity জানা থাকলে `Plan − Actual`।

হিসাবের আগে প্রয়োজনীয় type, unit, date/time ও business rule যাচাই করো। রাত পেরোনো duration-এ শুধু clock time বাদ দিলে ভুল হতে পারে; প্রয়োজনীয় সম্পূর্ণ timestamp ব্যবহার করো।

Left অজানা হলে duration null ও status দাও। শুধু arrival জানা থেকে গাড়ি এখনও yard-এ আছে ধরে নেবে না। Missing মানে zero ধরে subtraction করবে না।

| পরিস্থিতি | বুঝবে কীভাবে |
|---|---|
| Technical error | Conversion/সূত্র কাজ করছে না; raw দেখে কারণ ঠিক করতে হবে |
| Business exception | Data বৈধ হলেও business target ভেঙেছে, যেমন verified late delivery |
| Unusual but valid | অস্বাভাবিক বড় মান, কিন্তু source দিয়ে নিশ্চিত; স্বয়ংক্রিয়ভাবে বাদ নয় |
| Metric-specific অসম্পূর্ণতা | Booking গোনা যাবে, কিন্তু Left ছাড়া duration জানা যাবে না |

**ক্রম:** হিসাবের inputs প্রস্তুত → calculation → ফলের business meaning পরীক্ষা।

**বাদ দেওয়া যাবে?** কোনো নতুন হিসাব বা flag প্রয়োজন না হলে।

## ধাপ ১১ — Row count, totals, keys এবং missing coverage মিলিয়ে দেখো

এখন যাচাই করবে পরিবর্তনের ফল লক্ষ্য অনুযায়ী সঠিক হয়েছে কি না। বড় পরিবর্তনের পরেও এই যাচাই করবে; শুধু একেবারে শেষে নয়।

- [ ] Row count যত বেড়েছে/কমেছে, তার কারণ জানা আছে।
- [ ] প্রয়োজনীয় key ও intended grain ঠিক আছে।
- [ ] Merge-এ অপ্রত্যাশিত multiplication বা unmatched record বোঝা হয়েছে।
- [ ] প্রয়োজনীয় totals মেলানো হয়েছে; পার্থক্যের ব্যাখ্যা আছে।
- [ ] Types ছাড়াও date, quantity, unit ও business rule পরীক্ষা করা হয়েছে।
- [ ] Missing/invalid-এর count, percentage ও কোন হিসাবে প্রভাব—লেখা আছে।
- [ ] কিছু original rows-এর সঙ্গে cleaned output হাতে মিলিয়ে দেখা হয়েছে।

Cleaning-এর আগে–পরে totals সবসময় সমান থাকবে না। Duplicate copy সরালে total কমার কথা; কত কমবে ও কেন কমবে তা মেলাতে হবে।

কোনো column ৯০% complete হলেই report গ্রহণযোগ্য—এমন স্থায়ী নিয়ম নেই। Missing অংশটি কোন সময়, carrier বা group-এ বেশি, এবং তোমার সিদ্ধান্তে কতটা প্রভাব ফেলে—সেটিও দেখবে।

**কখন এগোবে?** যে হিসাব রিপোর্ট করছ তার জন্য data যথেষ্ট নির্ভরযোগ্য, checks সন্তোষজনক, এবং unresolved limitations স্পষ্ট হলে। Critical তথ্য অপর্যাপ্ত হলে source খুঁজবে বা প্রশ্নের scope সীমিত করবে।

**বাদ দেওয়া যাবে?** Output ব্যবহার করার আগে যথাযথ validation প্রয়োজন।

## ধাপ ১২ — সিদ্ধান্ত লিখে রাখো, output load করো এবং refresh যাচাই করো

Documentation কাজের সঙ্গে সঙ্গে লিখবে; এই ধাপে গুছিয়ে হস্তান্তর করবে।

| Issue | কত records | সিদ্ধান্ত ও কারণ | অবস্থা |
|---|---:|---|---|
| Repeated export copy | ১টি অতিরিক্ত copy | একই booking বলে যাচাই; একটি রাখা | Resolved |
| Actual অজানা | ১ booking | null রাখা; source যাচাই প্রয়োজন | Open |

Output-এর স্পষ্ট headers ও column meanings রাখো। প্রয়োজন হলে cleaned data, exception list ও সংক্ষিপ্ত change log আলাদা করো। Excel helper ফলকে স্থায়ী করতে চাইলে **ফল যাচাই → Paste Values → referenced working column বাদ**; raw copy থাকবে।

পুনরাবৃত্ত Power Query কাজে নতুন compatible file দিয়ে Refresh চালাও। Expected নতুন period/rows এসেছে, totals মিলেছে, নতুন errors/duplicates হয়নি—দেখো। File রাখলেই সব ব্যবস্থায় নিজে থেকে refresh হবে ধরে নিও না। Successful refresh একা সঠিক output প্রমাণ করে না। [১১]

একবারের কাজ হলে recurring refresh setup দরকার নেই। তবু কী করেছ, কেন করেছ, কী অসম্পূর্ণ—এই তথ্য দেবে।

## একই ছোট উদাহরণে পুরো পথ

লক্ষ্য: আলাদা booking গোনা, Plan pallets যোগ করা এবং জানা Actual জানানো। নিচে মূল গাইডের transport উদাহরণের কয়েকটি column নেওয়া হয়েছে। উদ্ধৃতির মধ্যে Text, `null` মানে অনুপস্থিত।

| SourceRow | Site | Booking_ID | Plan | Actual |
|---|---|---|---|---|
| 1 | LEI | null | null | null |
| 2 | null | `" 001 "` | `"10"` | `"8"` |
| 3 | null | `"001"` | `"10"` | `"8"` |
| 4 | null | `"002"` | `"12"` | `" N/A "` |
| 5 | null | null | null | null |

**জানা নিয়ম:** Row 1 Site-heading; rows 2–4 ওই Site-এর এবং order ঠিক। Row 5 শুধু spacer, boundary নয়। Rows 2 ও 3 একই booking-এর নিশ্চিত repeated copy। ID-এর বাইরের space অর্থহীন, zero গুরুত্বপূর্ণ। Actual-এর N/A মানে অজানা quantity।

| কোন ধাপ | এই data-তে কী করলে | Row count |
|---|---|---:|
| ১–৩ | লক্ষ্য/grain/key বুঝলে; raw রাখলে; ID Text রক্ষা ও inspection করলে | ৫ |
| ৪ | নিশ্চিত spacer row 5 সরালে | ৪ |
| ৪ | Site Fill Down করে rows 2–4-এ LEI নিলে | ৪ |
| ৪ | এখন heading row 1 সরালে | ৩ |
| ৫ | Split প্রয়োজন নেই; এগিয়ে গেলে | ৩ |
| ৬ | ID-এর বাইরের space Trim করলে | ৩ |
| ৭ | Actual marker Trim করে null; Plan/Actual Number করলে | ৩ |
| ৮ | SourceRow-কে business comparison-এ না ধরে নিশ্চিত repeated copy একটি সরালে | ২ |
| ৯ | অন্য table বা reshape দরকার নেই; এগিয়ে গেলে | ২ |
| ১০–১২ | Missing status, totals/coverage, validation ও change log তৈরি করলে | ২ |

Final table:

| Site | Booking_ID | Plan | Actual |
|---|---|---:|---:|
| LEI | `001` | 10 | 8 |
| LEI | `002` | 12 | null |

ফল:

- Booking = **২টি**।
- Plan = **২২ pallets**।
- জানা Actual = **৮ pallets**; সম্পূর্ণ Actual total জানা নেই।
- Actual missing = **১/২ × ১০০ = ৫০%** booking।
- Duplicate বাদ দেওয়ার আগে Plan ৩২ ছিল; অতিরিক্ত copy-এর ১০ বাদ গিয়ে ২২ হওয়া ঠিক।
- `২২ − ৮ = ১৪`-কে নিশ্চিত shortfall বলা যাবে না; একটি booking-এর Actual অজানা।
- Booking 002 count ও Plan total-এ ব্যবহারযোগ্য; সম্পূর্ণ Actual-ভিত্তিক হিসাবের জন্য অসম্পূর্ণ।

Rows 2 ও 3 যে একই booking-এর উৎস, সেই trace raw/change log-এ রাখবে। শুধু SourceRow আলাদা দেখে দুটি business record ধরবে না।

## কোন ক্রম অবশ্যই মানবে—এক নজরে

এগুলো আলাদা নির্ভরতা; একটি দীর্ঘ বাধ্যতামূলক button sequence নয়।

| আগে | পরে | কারণ |
|---|---|---|
| গুরুত্বপূর্ণ ID-কে Text হিসেবে import করার নিয়ম ঠিক করা | Import/স্বয়ংক্রিয় type conversion | হারানো zero পরে শুধু type বদলে ফেরে না |
| Heading-এর context উদ্ধার | Heading বাদ | কোন record কোন group-এর তা রক্ষা |
| Group, order ও inheritance rule নিশ্চিত | Fill Down | অন্য record/group-এর মান বসানো ঠেকানো |
| Split করলে তৈরি নতুন boundaries দেখা | প্রয়োজনীয় Trim | নতুন বাইরের space ধরতে |
| Missing marker-এর অর্থ ও source date/number format বোঝা | Parsing/conversion | অনুমানভিত্তিক value ঠেকানো |
| তুলনার দরকারি normalization ও grain/retention rule | Matching/deduplication | সঠিক record মিলানো ও রাখা |
| Formula output যাচাই ও স্থায়ী values করা, যদি সেটিই উদ্দেশ্য | Referenced working column বাদ | Formula reference ভেঙে যাওয়া ঠেকানো |
| প্রয়োজনীয় inputs প্রস্তুত | Calculation | Unknown বা ভুল type দিয়ে ভুল হিসাব ঠেকানো |
| Validation ও limitations স্পষ্ট করা | Report বিশ্বাস করে ব্যবহার | ফলের যথার্থতা বোঝা |

## কোন কাজ আগে–পরে করা যায়

**শর্ত: এক কাজ অন্যটির input, scope, group, order বা দরকারি তথ্য বদলাবে না।**

| উদাহরণ | কেন বদলাতে পারে |
|---|---|
| Booking_ID Trim ↔ Carrier uppercase | আলাদা স্বাধীন fields; দুটির অর্থহীন পার্থক্য সংশোধন |
| Booking_ID Trim ↔ Site Fill Down | ID পরিবর্তনে Site grouping/order বদলায় না—এই উদাহরণে |
| Date parse ↔ Plan Number conversion | প্রয়োজনীয় নিয়ম জানা থাকলে আলাদা fields-এর independent conversion |
| Rename/Reorder আগে বা পরে | পরের steps-এর নাম/position references ঠিক রাখা হলে |
| নিশ্চিত exact copy আগে বাদ | Copy জানা ও দরকারি তথ্য না হারালে; normalized comparison দরকার হলে তার প্রস্তুতি আগে |

Append/Merge, Filter/Sort, Fill Down বা Unpivot-কে সবসময় স্বাধীন ধরে নেবে না। বিশেষ করে row order, grain, group scope বা aggregation বদলালে ফলও বদলাতে পারে।

## কোন ধাপের পরিবর্তন বাদ দেওয়া যায়

| ধাপ | কখন পরিবর্তন লাগবে না |
|---|---|
| ৪ | Context, header ও layout ঠিক |
| ৫ | দরকারি fields আগে থেকেই আলাদা |
| ৬ | প্রয়োজনীয় text/key/category আগে থেকেই পরিষ্কার |
| ৭ | Missing handling প্রয়োজন নেই এবং values/types সঠিক |
| ৮ | Duplicate/revision সমস্যা নেই |
| ৯ | অন্য table বা reshape দরকার নেই |
| ১০ | নতুন measure/flag দরকার নেই |

ধাপ ১–৩-এর প্রস্তুতি, ধাপ ১১-এর যাচাই এবং ধাপ ১২-এর সিদ্ধান্ত/সীমাবদ্ধতা জানানোর উদ্দেশ্য বজায় রাখবে। আগে সম্পন্ন নির্ভরযোগ্য কাজ পুনরায় না করেও তার ফল ব্যবহার করা যায়। Recurring refresh কেবল পুনরাবৃত্ত workflow-তে প্রয়োজন।

AutoFit, রং/font বদলানো, chart সাজানো, REPT দিয়ে bar বানানো, Word-এ নিয়ে manual extraction এবং N/T-এর বিশেষ ব্যবহার—এসব প্রয়োজনভিত্তিক অতিরিক্ত কৌশল। Core cleaning-এর সব ফাইলে এগুলো লাগবে না।

## অনুশীলনের জন্য প্রয়োজনীয় commands ও formulas

এগুলো কাজ করার উপায়। কোনটি লাগবে, তা আগের ধাপের সিদ্ধান্ত দিয়ে বেছে নেবে। Formula-তে cell references নিজের data অনুযায়ী বদলাবে; Excel-এর regional setting অনুযায়ী argument separator আলাদা হতে পারে।

| কাজ | Excel-এ উদাহরণ | Power Query-তে সংশ্লিষ্ট command |
|---|---|---|
| Text-এর space ঠিক করা | Helper column-এ `=TRIM(A2)` | Text column নির্বাচন → Format → Trim |
| Case এক করা | `=UPPER(B2)`; নামের জন্য প্রয়োজনমতো `=PROPER(B2)` | Format → UPPERCASE/Capitalize Each Word |
| Length দেখা | `=LEN(A2)` | Add Column → Extract → Length |
| Numeric type পরীক্ষা | `=ISNUMBER(D2)` | Column type ও profiling দেখা |
| জানা numeric text convert করা | `=VALUE(D2)`; error হলে raw ও source format দেখবে | Data Type → উপযুক্ত Number; প্রয়োজন হলে Using Locale |
| জানা source format-এর date parse করা | Text to Columns → Date → source-এর DMY/MDY/YMD | Change Type → Using Locale; mixed source format হলে আলাদা parsing rule |
| Heading-এর মান নিচে বহন | যাচাইকৃত group-এর blanks-এ আগের প্রযোজ্য মানের reference | Fill → Down; group boundary রক্ষা করতে হবে |
| Code ও description আলাদা করা | Text to Columns; খালি destination ও ID-এর Text type | Split Column → By Delimiter |
| নিশ্চিত duplicate copy সরানো | Data → Remove Duplicates; comparison columns নির্বাচন | Comparison columns নির্বাচন → Remove Duplicates |

**শুরুতে এই পাঁচটি হাতে করো:** `TRIM`, `UPPER`, `LEN`, `ISNUMBER`, `VALUE`। তারপর একই কাজ Power Query-তে করো। `TRIM`/`UPPER` সব field-এ অন্ধভাবে চালাবে না; ধাপ ৬-এর শর্তগুলো প্রযোজ্য। `PROPER` acronym বা বিশেষ নামের বানান বদলাতে পারে।

Power Query-র Applied Steps-এ প্রতিটি বড় পরিবর্তনের পরে preview, row count ও প্রয়োজনীয় totals দেখো। ভুল হলে সংশ্লিষ্ট step ঠিক করো; raw data হাতে পাল্টে সমস্যাটি আড়াল কোরো না।

## কাজের সময় ব্যবহার করার ছোট checklist

- [ ] লক্ষ্য, scope, grain ও key জানা আছে।
- [ ] Raw, গুরুত্বপূর্ণ IDs এবং context সুরক্ষিত।
- [ ] সমস্যা ও baseline দেখা হয়েছে; প্রযোজ্য operations বেছে নিয়েছি।
- [ ] দরকারি context উদ্ধার ছাড়া row/column মুছিনি।
- [ ] অনুমোদিত text/category/missing/type rules প্রয়োগ করেছি।
- [ ] Duplicate, revision ও বৈধ detail আলাদা করেছি।
- [ ] Join/reshape-এর পরে grain, keys ও coverage পরীক্ষা করেছি—যদি প্রযোজ্য হয়।
- [ ] Calculation-এ unknown-কে zero ধরিনি।
- [ ] Counts, totals, business rules ও affected percentages মিলিয়েছি।
- [ ] কী করেছি, কেন করেছি, কী অসম্পূর্ণ এবং refresh কীভাবে যাচাই করেছি—লিখেছি।

কাজের ক্রম নিয়ে সন্দেহ হলে জিজ্ঞেস করবে: **“এই কাজের জন্য অন্য কাজের ফল দরকার?”** এবং **“এটি আগে করলে দরকারি তথ্য হারাবে?”** কোনোটি হ্যাঁ হলে সেই নির্ভরতা মেনে চলবে।

## উৎস ও যাচাইয়ের সীমা

ব্যবহারকারীর দেওয়া চারটি transcript: CLEAN framework; Deborah Ashby-এর Excel/Power Query webinar; US presidents data দিয়ে Excel cleaning tutorial; Yoda Learning-এর cleaning tricks compilation। সঙ্গে দেওয়া Power_Query_Concise_Guide_Bangla.md ও আলোচনার উদাহরণ ব্যবহার করা হয়েছে। ভিডিওর সব দাবি অপরিবর্তিত নেওয়া হয়নি; শর্তসাপেক্ষ ও ভুল সাধারণীকরণ সংশোধন করা হয়েছে।

এই পাঠ একটি সিদ্ধান্ত ও অনুশীলন-সহায়ক গাইড। এই পাঠের উদাহরণের arithmetic ও row counts যাচাই করা হয়েছে; এটি Power Query Editor-এ চালানো workbook নয়।

১. [Microsoft: Power Query profiling](https://learn.microsoft.com/en-us/power-query/data-profiling-tools)
২. [Microsoft: IS functions](https://support.microsoft.com/en-us/excel/functions/is-functions)
৩. [Microsoft: Fill values](https://learn.microsoft.com/en-us/power-query/fill-values-column)
৪. [Microsoft: Go To Special selections](https://support.microsoft.com/en-us/excel/find-and-select-cells-that-meet-specific-conditions-in-excel)
৫. [Excel TRIM](https://support.microsoft.com/en-us/excel/functions/trim-function), [Power Query Text.Trim](https://learn.microsoft.com/en-us/powerquery-m/text-trim), [Text.Clean](https://learn.microsoft.com/en-us/powerquery-m/text-clean)
৬. [Microsoft: Find and Replace/wildcards](https://support.microsoft.com/en-US/Excel/get-started/find-or-replace-text-and-numbers-on-a-worksheet)
৭. [Date.FromText](https://learn.microsoft.com/en-us/powerquery-m/date-fromtext), [Text Import Wizard](https://support.microsoft.com/en-us/excel/text-import-wizard), [DATEVALUE](https://support.microsoft.com/en-us/excel/functions/datevalue-function)
৮. [Number formats](https://support.microsoft.com/en-us/excel/get-started/available-number-formats-in-excel), [TEXT function](https://support.microsoft.com/en-us/excel/functions/text-function)
৯. [Microsoft: Power Query duplicate handling](https://learn.microsoft.com/en-us/power-query/working-with-duplicates)
১০. [Microsoft: Table.Unpivot](https://learn.microsoft.com/en-us/powerquery-m/table-unpivot)
১১. [Microsoft: Folder combine and refresh](https://support.microsoft.com/en-us/excel/import-data-from-a-folder-with-multiple-files-power-query)
