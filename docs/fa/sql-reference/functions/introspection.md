---
machine_translated: true
machine_translated_rev: d734a8e46ddd7465886ba4133bff743c55190626
toc_priority: 65
toc_title: "\u062F\u0631\u0648\u0646 \u0646\u06AF\u0631\u06CC"
---

# توابع درون گرایی {#introspection-functions}

شما می توانید توابع شرح داده شده در این فصل به درون نگری استفاده کنید [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) و [DWARF](https://en.wikipedia.org/wiki/DWARF) برای پروفایل پرس و جو.

!!! warning "اخطار"
    این توابع کند هستند و ممکن است ملاحظات امنیتی تحمیل کنند.

برای بهره برداری مناسب از توابع درون گرایی:

-   نصب `clickhouse-common-static-dbg` بسته

-   تنظیم [اجازه دادن به \_فعال کردن اختلال در عملکرد](../../operations/settings/settings.md#settings-allow_introspection_functions) تنظیم به 1.

        For security reasons introspection functions are disabled by default.

تاتر موجب صرفه جویی در گزارش نیمرخ به [\_قطع](../../operations/system-tables.md#system_tables-trace_log) جدول سیستم. اطمینان حاصل کنید که جدول و پیشفیلتر به درستی پیکربندی شده است.

## افزودن مدخل جدید {#addresstoline}

تبدیل آدرس حافظه مجازی در داخل clickhouse فرایند سرور به نام فایل و شماره خط در clickhouse کد منبع.

اگر شما استفاده از بسته های رسمی تاتر, شما نیاز به نصب `clickhouse-common-static-dbg` بسته

**نحو**

``` sql
addressToLine(address_of_binary_instruction)
```

**پارامترها**

-   `address_of_binary_instruction` ([UInt64](../../sql-reference/data-types/int-uint.md)) — Address of instruction in a running process.

**مقدار بازگشتی**

-   نام فایل کد منبع و شماره خط در این فایل حد و مرز مشخصی توسط روده بزرگ.

        For example, `/build/obj-x86_64-linux-gnu/../dbms/Common/ThreadPool.cpp:199`, where `199` is a line number.

-   نام یک باینری, اگر تابع می تواند اطلاعات اشکال زدایی پیدا کنید.

-   رشته خالی, اگر نشانی معتبر نیست.

نوع: [رشته](../../sql-reference/data-types/string.md).

**مثال**

فعال کردن توابع درون گرایی:

``` sql
SET allow_introspection_functions=1
```

انتخاب رشته اول از `trace_log` جدول سیستم:

``` sql
SELECT * FROM system.trace_log LIMIT 1 \G
```

``` text
Row 1:
──────
event_date:              2019-11-19
event_time:              2019-11-19 18:57:23
revision:                54429
timer_type:              Real
thread_number:           48
query_id:                421b6855-1858-45a5-8f37-f383409d6d72
trace:                   [140658411141617,94784174532828,94784076370703,94784076372094,94784076361020,94784175007680,140658411116251,140658403895439]
```

این `trace` درست شامل ردیابی پشته در حال حاضر نمونه برداری.

گرفتن نام فایل کد منبع و شماره خط برای یک نشانی واحد:

``` sql
SELECT addressToLine(94784076370703) \G
```

``` text
Row 1:
──────
addressToLine(94784076370703): /build/obj-x86_64-linux-gnu/../dbms/Common/ThreadPool.cpp:199
```

استفاده از تابع به ردیابی کل پشته:

``` sql
SELECT
    arrayStringConcat(arrayMap(x -> addressToLine(x), trace), '\n') AS trace_source_code_lines
FROM system.trace_log
LIMIT 1
\G
```

این [اررایماپ](higher-order-functions.md#higher_order_functions-array-map) تابع اجازه می دهد تا برای پردازش هر عنصر منحصر به فرد از `trace` تنظیم توسط `addressToLine` تابع. در نتیجه این پردازش می بینید در `trace_source_code_lines` ستون خروجی.

``` text
Row 1:
──────
trace_source_code_lines: /lib/x86_64-linux-gnu/libpthread-2.27.so
/usr/lib/debug/usr/bin/clickhouse
/build/obj-x86_64-linux-gnu/../dbms/Common/ThreadPool.cpp:199
/build/obj-x86_64-linux-gnu/../dbms/Common/ThreadPool.h:155
/usr/include/c++/9/bits/atomic_base.h:551
/usr/lib/debug/usr/bin/clickhouse
/lib/x86_64-linux-gnu/libpthread-2.27.so
/build/glibc-OTsEL5/glibc-2.27/misc/../sysdeps/unix/sysv/linux/x86_64/clone.S:97
```

## افزودن موقعیت {#addresstosymbol}

تبدیل آدرس حافظه مجازی در داخل clickhouse سرور روند به نمادی از clickhouse شی فایل های.

**نحو**

``` sql
addressToSymbol(address_of_binary_instruction)
```

**پارامترها**

-   `address_of_binary_instruction` ([UInt64](../../sql-reference/data-types/int-uint.md)) — Address of instruction in a running process.

**مقدار بازگشتی**

-   نمادی از clickhouse شی فایل های.
-   رشته خالی, اگر نشانی معتبر نیست.

نوع: [رشته](../../sql-reference/data-types/string.md).

**مثال**

فعال کردن توابع درون گرایی:

``` sql
SET allow_introspection_functions=1
```

انتخاب رشته اول از `trace_log` جدول سیستم:

``` sql
SELECT * FROM system.trace_log LIMIT 1 \G
```

``` text
Row 1:
──────
event_date:    2019-11-20
event_time:    2019-11-20 16:57:59
revision:      54429
timer_type:    Real
thread_number: 48
query_id:      724028bf-f550-45aa-910d-2af6212b94ac
trace:         [94138803686098,94138815010911,94138815096522,94138815101224,94138815102091,94138814222988,94138806823642,94138814457211,94138806823642,94138814457211,94138806823642,94138806795179,94138806796144,94138753770094,94138753771646,94138753760572,94138852407232,140399185266395,140399178045583]
```

این `trace` درست شامل ردیابی پشته در حال حاضر نمونه برداری.

گرفتن نماد برای یک نشانی واحد:

``` sql
SELECT addressToSymbol(94138803686098) \G
```

``` text
Row 1:
──────
addressToSymbol(94138803686098): _ZNK2DB24IAggregateFunctionHelperINS_20AggregateFunctionSumImmNS_24AggregateFunctionSumDataImEEEEE19addBatchSinglePlaceEmPcPPKNS_7IColumnEPNS_5ArenaE
```

استفاده از تابع به ردیابی کل پشته:

``` sql
SELECT
    arrayStringConcat(arrayMap(x -> addressToSymbol(x), trace), '\n') AS trace_symbols
FROM system.trace_log
LIMIT 1
\G
```

این [اررایماپ](higher-order-functions.md#higher_order_functions-array-map) تابع اجازه می دهد تا برای پردازش هر عنصر منحصر به فرد از `trace` تنظیم توسط `addressToSymbols` تابع. نتیجه این پردازش شما در دیدن `trace_symbols` ستون خروجی.

``` text
Row 1:
──────
trace_symbols: _ZNK2DB24IAggregateFunctionHelperINS_20AggregateFunctionSumImmNS_24AggregateFunctionSumDataImEEEEE19addBatchSinglePlaceEmPcPPKNS_7IColumnEPNS_5ArenaE
_ZNK2DB10Aggregator21executeWithoutKeyImplERPcmPNS0_28AggregateFunctionInstructionEPNS_5ArenaE
_ZN2DB10Aggregator14executeOnBlockESt6vectorIN3COWINS_7IColumnEE13immutable_ptrIS3_EESaIS6_EEmRNS_22AggregatedDataVariantsERS1_IPKS3_SaISC_EERS1_ISE_SaISE_EERb
_ZN2DB10Aggregator14executeOnBlockERKNS_5BlockERNS_22AggregatedDataVariantsERSt6vectorIPKNS_7IColumnESaIS9_EERS6_ISB_SaISB_EERb
_ZN2DB10Aggregator7executeERKSt10shared_ptrINS_17IBlockInputStreamEERNS_22AggregatedDataVariantsE
_ZN2DB27AggregatingBlockInputStream8readImplEv
_ZN2DB17IBlockInputStream4readEv
_ZN2DB26ExpressionBlockInputStream8readImplEv
_ZN2DB17IBlockInputStream4readEv
_ZN2DB26ExpressionBlockInputStream8readImplEv
_ZN2DB17IBlockInputStream4readEv
_ZN2DB28AsynchronousBlockInputStream9calculateEv
_ZNSt17_Function_handlerIFvvEZN2DB28AsynchronousBlockInputStream4nextEvEUlvE_E9_M_invokeERKSt9_Any_data
_ZN14ThreadPoolImplI20ThreadFromGlobalPoolE6workerESt14_List_iteratorIS0_E
_ZZN20ThreadFromGlobalPoolC4IZN14ThreadPoolImplIS_E12scheduleImplIvEET_St8functionIFvvEEiSt8optionalImEEUlvE1_JEEEOS4_DpOT0_ENKUlvE_clEv
_ZN14ThreadPoolImplISt6threadE6workerESt14_List_iteratorIS0_E
execute_native_thread_routine
start_thread
clone
```

## درهم و برهم کردن {#demangle}

تبدیل یک نماد است که شما می توانید با استفاده از [افزودن موقعیت](#addresstosymbol) تابع به ج++ نام تابع.

**نحو**

``` sql
demangle(symbol)
```

**پارامترها**

-   `symbol` ([رشته](../../sql-reference/data-types/string.md)) — Symbol from an object file.

**مقدار بازگشتی**

-   نام تابع ج++.
-   رشته خالی اگر یک نماد معتبر نیست.

نوع: [رشته](../../sql-reference/data-types/string.md).

**مثال**

فعال کردن توابع درون گرایی:

``` sql
SET allow_introspection_functions=1
```

انتخاب رشته اول از `trace_log` جدول سیستم:

``` sql
SELECT * FROM system.trace_log LIMIT 1 \G
```

``` text
Row 1:
──────
event_date:    2019-11-20
event_time:    2019-11-20 16:57:59
revision:      54429
timer_type:    Real
thread_number: 48
query_id:      724028bf-f550-45aa-910d-2af6212b94ac
trace:         [94138803686098,94138815010911,94138815096522,94138815101224,94138815102091,94138814222988,94138806823642,94138814457211,94138806823642,94138814457211,94138806823642,94138806795179,94138806796144,94138753770094,94138753771646,94138753760572,94138852407232,140399185266395,140399178045583]
```

این `trace` زمینه شامل ردیابی پشته در لحظه نمونه برداری.

گرفتن نام تابع برای یک نشانی واحد:

``` sql
SELECT demangle(addressToSymbol(94138803686098)) \G
```

``` text
Row 1:
──────
demangle(addressToSymbol(94138803686098)): DB::IAggregateFunctionHelper<DB::AggregateFunctionSum<unsigned long, unsigned long, DB::AggregateFunctionSumData<unsigned long> > >::addBatchSinglePlace(unsigned long, char*, DB::IColumn const**, DB::Arena*) const
```

استفاده از تابع به ردیابی کل پشته:

``` sql
SELECT
    arrayStringConcat(arrayMap(x -> demangle(addressToSymbol(x)), trace), '\n') AS trace_functions
FROM system.trace_log
LIMIT 1
\G
```

این [اررایماپ](higher-order-functions.md#higher_order_functions-array-map) تابع اجازه می دهد تا برای پردازش هر عنصر منحصر به فرد از `trace` تنظیم توسط `demangle` تابع. در نتیجه این پردازش می بینید در `trace_functions` ستون خروجی.

``` text
Row 1:
──────
trace_functions: DB::IAggregateFunctionHelper<DB::AggregateFunctionSum<unsigned long, unsigned long, DB::AggregateFunctionSumData<unsigned long> > >::addBatchSinglePlace(unsigned long, char*, DB::IColumn const**, DB::Arena*) const
DB::Aggregator::executeWithoutKeyImpl(char*&, unsigned long, DB::Aggregator::AggregateFunctionInstruction*, DB::Arena*) const
DB::Aggregator::executeOnBlock(std::vector<COW<DB::IColumn>::immutable_ptr<DB::IColumn>, std::allocator<COW<DB::IColumn>::immutable_ptr<DB::IColumn> > >, unsigned long, DB::AggregatedDataVariants&, std::vector<DB::IColumn const*, std::allocator<DB::IColumn const*> >&, std::vector<std::vector<DB::IColumn const*, std::allocator<DB::IColumn const*> >, std::allocator<std::vector<DB::IColumn const*, std::allocator<DB::IColumn const*> > > >&, bool&)
DB::Aggregator::executeOnBlock(DB::Block const&, DB::AggregatedDataVariants&, std::vector<DB::IColumn const*, std::allocator<DB::IColumn const*> >&, std::vector<std::vector<DB::IColumn const*, std::allocator<DB::IColumn const*> >, std::allocator<std::vector<DB::IColumn const*, std::allocator<DB::IColumn const*> > > >&, bool&)
DB::Aggregator::execute(std::shared_ptr<DB::IBlockInputStream> const&, DB::AggregatedDataVariants&)
DB::AggregatingBlockInputStream::readImpl()
DB::IBlockInputStream::read()
DB::ExpressionBlockInputStream::readImpl()
DB::IBlockInputStream::read()
DB::ExpressionBlockInputStream::readImpl()
DB::IBlockInputStream::read()
DB::AsynchronousBlockInputStream::calculate()
std::_Function_handler<void (), DB::AsynchronousBlockInputStream::next()::{lambda()#1}>::_M_invoke(std::_Any_data const&)
ThreadPoolImpl<ThreadFromGlobalPool>::worker(std::_List_iterator<ThreadFromGlobalPool>)
ThreadFromGlobalPool::ThreadFromGlobalPool<ThreadPoolImpl<ThreadFromGlobalPool>::scheduleImpl<void>(std::function<void ()>, int, std::optional<unsigned long>)::{lambda()#3}>(ThreadPoolImpl<ThreadFromGlobalPool>::scheduleImpl<void>(std::function<void ()>, int, std::optional<unsigned long>)::{lambda()#3}&&)::{lambda()#1}::operator()() const
ThreadPoolImpl<std::thread>::worker(std::_List_iterator<std::thread>)
execute_native_thread_routine
start_thread
clone
```
