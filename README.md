# Base_GSM_TIP
Base variant GSM TIP model  
__Цель - привести ГСМ ТИП в соответствие с исправлениями, выполненными в процессе создания EAGLE.__

При объединении HAMMONIA и ГСМ ТИП наиболее перспективной показалась идея “перекрытия” высотных областей модели от 80 до 120 км. Сильная изменчивость параметров HAMMONIA приводит к большим вертикальным скоростям в ГСМ ТИП в нижней термосфере, а точнее к сильной нерегулярности вертикального профиля Vr. В результате возникают проблемы с составом (особенно n(O)). Данную проблему удалось (надеюсь) разрешить используя медианное сглаживание вертикального профиля  Vr (veter_ham, amed3) и переделкой divVr (o2pro).  
Другие замеченные проблемы, связанные с особенностью Intel-компилятора:
- ошибки, возникющие в программе интерполяции, приводящие к появлению NAN в inter2 
- в программе setka_kl
- ошибки “исчезновение порядка” из-за меньшего порядка величин, которые “переваривает” Intel (например, E-60 в Microsoft и E-30 Intel). Часто такие ошибки также приводили к NAN, что затрудняло их поиск (chep, chept, noprog,nprog)  
__Перечень замеченных проблем, которые требуют фиксации:__
- запись в file4 концентраций NO+ и O2+ (можно заменить горячий кислород - температуру и концентрацию);
корректировать поток диссоциирующего излучения, в том числе брать La из ионизирующей части спектра. 

__Сделанные изменения:__
1. Добавлен расчет Vr в 1-ой точке (80 км)
2. Переделан noprog - div(nVr) с учетом знака Vr
3. Медианное сглаживание вертикального профиля Vr
4. Стандартный вариант О2 , но член div(nVr) записан с учетом знака Vr
5. Подготовлены физические константы и сведены в один модуль mo_bas_gsm.f90
6. Удален расчет Ohot, вместо него в file4 пишутся концентрации O2+ и NO+ (18 и 19 параметры)
Для создания начальных значений в f4 (фотохимическое приближение) следует запустить модель с расчетом NO (mass(21)=1) и суммарным мол. ионом (mass(20)=0)
7. МСИС-90 (gtd6) заменен на MSIS-2000 (gtd7). В danmodel по параметру mass(17) выбрать вариант MSIS не получится - всегда MSIS2000
Из-за совпадения имен (spline) в MSIS2000 и ГСМ ТИП, spline в ГСМ ТИП переименован в spline_bas. Изменение МСИС90 на МСИС2000 на нг дает изменение на высотах F2-области (проведены сравнительные расчеты) 
8. В массив Solu(1), который отвечает за La в диссоциирующей части спектра (UV), пишется sole(nse), тот же La, но в ионизирующей части (EUV).  
9. Заменен расчет скорости диссоциации, теперь поглощение считается как в ionizu. Значения скорости диссоциации сохраняется 
в массив qdis(2,its,ids,nh) и используется в нагреве (индекс qdis(2,...)) и составе (индекс qdis(1,...))
10. Восстановлена зависимость потенциала через полярную шапку от Кр при задании высыпаний Zhang&Paxton (cyclt1)
11. fac2 задается в danmodel (не рассчитывается!) и равен в невозмущенных условиях 9.0e-13
12. Расчет N(2D) в фотохим. постановке заменен на расчет в диф. постановке (ndprog)
13. Ионизация NO электронными высыпаниями не учитывается (правильно)
14. В alpha.inc: к-т alyam6 изменен с 0 на 0.5 (фотодиссоциация N2)
                 добавлена реакция N2+ + e
15. В wwt, wws и inpout избавился от goto.

__Ветка EDDY - переход к 2D к-ту турбулентной диффузии__
1. turbko - расчет 2мерного к-та турбулентной диффузии. При задании значений используется основная идея - 
   при равенстве Kt = Kmol, Kt=0. Профиль задается как и раньше - по Shimazaki
2. Все программы, в которые передается Kt переделаны. 
