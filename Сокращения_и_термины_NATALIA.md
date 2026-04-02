# Сокращения, обозначения и термины ПО НА NATALIA

## 1. Назначение документа

Настоящий документ содержит перечень сокращений, обозначений, английских идентификаторов
и терминов, используемых в документации по алгоритму штатного ПО НА NATALIA.

Документ предназначен для:
- единообразного использования терминов во всех algorithm-файлах;
- уменьшения неоднозначности в описании режимов, команд и событий;
- подготовки итоговой версии документации в LaTeX.

---

## 2. Общие сокращения

| Сокращение | Расшифровка | Комментарий |
|---|---|---|
| НА | Научная аппаратура | В данном проекте — гамма-спектрометр |
| ПУ | Плата управления | Основной вычислительный и управляющий модуль |
| ПЭД | Плата электроники детекторов | Модуль детекторной электроники |
| БД | Блок детекторов | Физический детекторный блок |
| БВС КА | Бортовая вычислительная система космического аппарата | Внешняя система управления |
| КА | Космический аппарат | Носитель НА |
| МК | Микроконтроллер | STM32L496 |
| ПО | Программное обеспечение | В данном проекте — штатное ПО ПУ |
| ШПО | Штатное программное обеспечение | Основной firmware-алгоритм |
| CAN | Controller Area Network | Интерфейс связи с БВС КА |
| UniCAN | Протокол обмена по CAN | Используется для КУ, КТ, ТС |
| USB | Universal Serial Bus | Интерфейс вывода данных |
| RTC | Real-Time Clock | Часы реального времени |
| QSPI | Quad SPI | Интерфейс работы с NAND |
| SPI | Serial Peripheral Interface | Интерфейс работы с MRAM и др. |
| I2C | Inter-Integrated Circuit | Интерфейс датчиков и ПЭД |
| GPIO | General Purpose Input/Output | Линии ввода-вывода |
| ADC | Analog-to-Digital Converter | Аналого-цифровой преобразователь |
| MRAM | Magnetoresistive RAM | Энергонезависимая память настроек и служебных данных |
| NAND | NAND Flash Memory | Энергонезависимая память научных данных |
| ППЗУ | Постоянное запоминающее устройство | В документации используется для NAND |
| ТС | Телеметрическое сообщение | Сообщение НА -> БВС КА |
| КУ | Команда управления | Сообщение БВС КА -> НА |
| КТ | Команда телеметрии | Сообщение БВС КА -> НА с внешними данными |
| НИ | Научная информация | Форматы и пакеты, записываемые в NAND |
| CRC | Cyclic Redundancy Check | Контрольная сумма |
| VBUS | Линия питания USB host | Используется для контроля наличия USB host |
| SWD | Serial Wire Debug | Интерфейс прошивки/отладки |
| USART | Universal Synchronous/Asynchronous Receiver-Transmitter | Отладочный интерфейс |
| DC/DC | DC-DC преобразователь | Узел питания |
| LSE | Low-Speed External oscillator | Низкочастотный кварц RTC |
| HSE | High-Speed External oscillator | Высокочастотный кварц МК |

---

## 3. Термины уровня алгоритма

| Термин | Значение | Комментарий |
|---|---|---|
| Режим | Верхнеуровневое состояние работы НА | Например: DUTY, OBSERVE, ALARM |
| Подсостояние | Внутреннее состояние внутри режима | Например: OBS_ENTER, OBS_ACTIVE |
| Событие | Внешний или внутренний триггер для обработки алгоритмом | Команда, тик RTC, авария и т.д. |
| Переход | Смена состояния или подсостояния | Выполняется по событию и условию |
| Действие | Операция, выполняемая алгоритмом | Включение питания, запись MRAM и т.д. |
| Целевая операция | Основная длительная операция режима | Стирание, тест, наблюдение, вывод |
| Safe-состояние | Безопасная конфигурация аппаратуры | Обычно с отключёнными ПЭД и NAND |
| Активный банк | Банк NAND, используемый в текущем режиме | NAND1 или NAND2 |
| Сеанс наблюдений | Непрерывный цикл работы в OBSERVE | Имеет идентификатор session_id |
| Номер режима наблюдений | Идентификатор внутренней конфигурации внутри сеанса | Инкрементируется при `CMD_OBSERVE_CTRL` |
| Формат НИ | Структурированный блок научной/служебной информации | События, Счётчики, Спектр и т.д. |
| Пакет НИ | Базовая единица хранения НИ в NAND | 2048 байт |
| Переполнение NAND | Ситуация, когда активный банк заполнен | Вызывает автозавершение OBSERVE |
| Маска аварий | Битовая маска включённых аварийных признаков | Хранится в MRAM |
| Аварийный статус | Битовое поле активных аварий | `AlarmStatus` |
| Замаскированный аварийный статус | `AlarmStatus AND AlarmMask` | `MaskedAlarm` |

---

## 4. Обозначения верхнеуровневых режимов

| Обозначение | Расшифровка | Значение |
|---|---|---|
| INIT | Initialization | Инициализация после подачи питания |
| DUTY | Duty mode | Дежурный режим |
| ERASE | Erase mode | Режим стирания ППЗУ |
| TEST | Test mode | Режим тестирования ППЗУ |
| OBSERVE | Observation mode | Режим наблюдений |
| DUMP | Dump mode | Режим вывода данных |
| ALARM | Alarm mode | Аварийный режим |
| SHUTDOWN | Shutdown mode | Режим выключения |

---

## 5. Обозначения подсостояний режимов

### 5.1 Подсостояния режима наблюдений

| Обозначение | Расшифровка | Значение |
|---|---|---|
| OBS_ENTER | Observe enter | Подготовка входа в режим наблюдений |
| OBS_ACTIVE | Observe active | Активная фаза наблюдений |
| OBS_EXIT_CMD | Observe exit by command | Выход по команде |
| OBS_EXIT_FULL | Observe exit by full NAND | Выход по переполнению NAND |
| OBS_EXIT_ALARM | Observe exit by alarm | Выход по аварии |

### 5.2 Подсостояния режима стирания

| Обозначение | Расшифровка | Значение |
|---|---|---|
| ERASE_ENTER | Erase enter | Подготовка входа |
| ERASE_START | Erase start | Запуск стирания |
| ERASE_WAIT | Erase wait | Ожидание завершения |
| ERASE_FINISH_OK | Erase finish ok | Штатное завершение |
| ERASE_FINISH_CMD | Erase finish by command | Завершение по команде |
| ERASE_FINISH_ALARM | Erase finish by alarm | Завершение по аварии |

### 5.3 Подсостояния режима тестирования

| Обозначение | Расшифровка | Значение |
|---|---|---|
| TEST_ENTER | Test enter | Подготовка входа |
| TEST_WRITE | Test write | Запись тестовых данных |
| TEST_READ | Test read | Чтение тестовых данных |
| TEST_COMPARE | Test compare | Сравнение и подсчёт ошибок |
| TEST_SAVE | Test save | Сохранение результатов в MRAM |
| TEST_FINISH_OK | Test finish ok | Штатное завершение |
| TEST_FINISH_CMD | Test finish by command | Завершение по команде |
| TEST_FINISH_ALARM | Test finish by alarm | Завершение по аварии |

### 5.4 Подсостояния режима вывода данных

| Обозначение | Расшифровка | Значение |
|---|---|---|
| DUMP_ENTER | Dump enter | Подготовка входа |
| DUMP_IF_PREP | Dump interface prepare | Подготовка интерфейса вывода |
| DUMP_READ | Dump read | Чтение пакета из NAND |
| DUMP_SEND | Dump send | Передача пакета |
| DUMP_CHECK | Dump check | Проверка завершения вывода |
| DUMP_FINISH_OK | Dump finish ok | Штатное завершение |
| DUMP_FINISH_CMD | Dump finish by command | Завершение по команде |
| DUMP_FINISH_ALARM | Dump finish by alarm | Завершение по аварии |

### 5.5 Подсостояния аварийного режима

| Обозначение | Расшифровка | Значение |
|---|---|---|
| ALARM_ENTER | Alarm enter | Вход в аварийный режим |
| ALARM_SAFE | Alarm safe | Приведение в safe-состояние |
| ALARM_WAIT | Alarm wait | Ожидание команд и изменения статуса |
| ALARM_EXIT_DUTY | Alarm exit to duty | Выход в дежурный режим |
| ALARM_EXIT_SHUTDOWN | Alarm exit to shutdown | Выход в режим выключения |

### 5.6 Подсостояния режима выключения

| Обозначение | Расшифровка | Значение |
|---|---|---|
| SHDN_ENTER | Shutdown enter | Вход в режим выключения |
| SHDN_STOP_OP | Shutdown stop operation | Останов текущей активности |
| SHDN_SAVE | Shutdown save | Сохранение служебных данных |
| SHDN_POWER_OFF | Shutdown power off | Отключение аппаратуры |
| SHDN_WAIT_POWER_OFF | Shutdown wait power off | Ожидание снятия питания |

---

## 6. Обозначения внешних команд управления

| Обозначение | Полное имя | Значение |
|---|---|---|
| CMD_TELEM_REQ | Command telemetry request | КУ 1, запрос телеметрии |
| CMD_STATUS_REQ | Command status request | КУ 2, запрос статуса |
| CMD_SET_TIME | Command set time | КУ 3, предустановка времени |
| CMD_OBSERVE_START | Command observe start | КУ 4, включение режима наблюдений |
| CMD_OBSERVE_CTRL | Command observe control | КУ 5, управление режимом наблюдений |
| CMD_DUTY | Command duty | КУ 6, переход в дежурный режим |
| CMD_DUMP | Command dump | КУ 7, вывод данных |
| CMD_SET_CFG | Command set configuration | КУ 8, задание настроек |
| CMD_ERASE | Command erase | КУ 9, стирание ППЗУ |
| CMD_TEST | Command test | КУ 10, тест ППЗУ |
| CMD_TEST_RESULT | Command test result request | КУ 11, запрос результатов теста |
| CMD_SHUTDOWN | Command shutdown | КУ 12, выключение |
| CMD_RESET_ALARM | Command reset alarm | КУ 13, сброс аварийного статуса |

---

## 7. Обозначения внешних команд телеметрии

| Обозначение | Полное имя | Значение |
|---|---|---|
| TLM_TIME_SYNC | Telemetry time sync | КТ 1, сверка времени |
| TLM_ORBIT | Telemetry orbit | КТ 2, параметры орбиты |
| TLM_ATTITUDE | Telemetry attitude | КТ 3, параметры ориентации |
| TLM_MAGFIELD | Telemetry magnetic field | КТ 4, геомагнитное поле |

---

## 8. Обозначения внутренних событий

| Обозначение | Расшифровка | Значение |
|---|---|---|
| E_BOOT | Boot event | Подача питания, старт ПО |
| E_INIT_DONE | Init done | Инициализация завершена успешно |
| E_INIT_FAIL | Init fail | Инициализация завершена с ошибкой |
| E_RTC_1HZ | RTC 1 Hz tick | Тик часов 1 Гц |
| E_PED_TRIGGER | PED trigger event | Приход триггера от ПЭД |
| E_NAND_FULL | NAND full | Переполнение активного банка NAND |
| E_ERASE_DONE | Erase done | Стирание завершено |
| E_TEST_DONE | Test done | Тест завершён |
| E_DUMP_DONE | Dump done | Вывод данных завершён |
| E_STATUS_CHANGED | Status changed | Событие, требующее автоматической выдачи ТС «Статус» |
| E_MASKED_ALARM_SET | Masked alarm set | `MaskedAlarm != 0` |
| E_MASKED_ALARM_CLEAR | Masked alarm clear | `MaskedAlarm == 0` |
| E_OP_FAIL | Operation fail | Ошибка длительной операции |
| E_CAN_TX_ERROR | CAN transmit error | Ошибка передачи ТС по UniCAN |
| E_CAN_TX_RETRY_EXHAUSTED | CAN retry exhausted | Повторы передачи исчерпаны |

---

## 9. Обозначения форматов научной информации

| Обозначение | Расшифровка | Значение |
|---|---|---|
| NI | Scientific information | Научная информация |
| NI packet | Scientific information packet | Пакет НИ, 2048 байт |
| Event format | Format "События" | Формат событий |
| Counters format | Format "Счетчики" | Формат счётчиков |
| Spectrum-1 | Format "Спектр-1" | Формат гистограммы без zero suppression |
| Spectrum-2 | Format "Спектр-2" | Формат гистограммы с zero suppression |
| Telemetry format | Format "Телеметрия" | Формат служебной телеметрии |
| Sync format | Format "Синхронизация" | Формат синхронизации времени |
| Orbit format | Format "Орбита" | Формат орбитальных параметров |
| Attitude format | Format "Ориентация" | Формат ориентации |
| MagField format | Format "Геомагнитное поле" | Формат геомагнитного поля |

---

## 10. Обозначения аварийных признаков

| Обозначение | Расшифровка | Значение |
|---|---|---|
| AlarmStatus | Аварийный статус НА | Битовое поле активных аварий |
| AlarmMask | Маска аварийного статуса | Битовое поле маски |
| MaskedAlarm | Замаскированный аварийный статус | `AlarmStatus AND AlarmMask` |
| ALARM_MC_Temp | MCU temperature alarm | Температура МК вне диапазона |
| ALARM_PU_Temp | PU temperature alarm | Температура ПУ вне диапазона |
| ALARM_PED_Temp | PED temperature alarm | Температура ПЭД вне диапазона |
| ALARM_BD_Temp | Detector block temperature alarm | Температура БД вне диапазона |
| ALARM_PU_Volt | PU voltage alarm | Напряжение ПУ вне диапазона |
| ALARM_PU_Curr | PU current alarm | Ток ПУ вне диапазона |
| ALARM_PED_Volt | PED voltage alarm | Напряжение ПЭД вне диапазона |
| ALARM_PED_Curr | PED current alarm | Ток ПЭД вне диапазона |
| ALARM_PED_PS | PED power supply alarm | Авария питания ПЭД |
| ALARM_PED_ST | PED status alarm | Ошибка статуса ПЭД |
| ALARM_NAND | NAND alarm | Авария банка ППЗУ |
| ALARM_USB | USB alarm | Авария интерфейса USB |

---

## 11. Обозначения статусных линий и сигналов

| Обозначение | Расшифровка | Значение |
|---|---|---|
| PU_CAN1_SHDN | CAN1 transceiver shutdown control | Управление трансивером CAN1 |
| PU_CAN1_S | CAN1 silent mode control | Режим прослушивания CAN1 |
| PU_CAN2_SHDN | CAN2 transceiver shutdown control | Управление трансивером CAN2 |
| PU_CAN2_S | CAN2 silent mode control | Режим прослушивания CAN2 |
| PU_NAND1_PS | NAND1 power switch control | Управление питанием NAND1 |
| PU_NAND1_PSON | NAND1 power status | Статус наличия питания NAND1 |
| PU_NAND2_PS | NAND2 power switch control | Управление питанием NAND2 |
| PU_NAND2_PSON | NAND2 power status | Статус наличия питания NAND2 |
| PU_PED_PS | PED power switch control | Управление питанием ПЭД |
| PU_USB_VBUS | USB VBUS status | Статус наличия VBUS |
| PED_TG | PED trigger | Триггер события от ПЭД |
| PED_TGRES | PED trigger reset | Сброс триггера ПЭД |
| PED_INHIBIT | PED inhibit | Запрет регистрации событий в ПЭД |
| PED_SLEEP | PED sleep | Режим пониженного энергопотребления ПЭД |
| PED_PSON | PED power status | Статус наличия питания на ПЭД |
| RTC_OUT | RTC 1 Hz output | Выход 1 Гц RTC |

---

## 12. Обозначения служебных переменных и данных

| Обозначение | Расшифровка | Значение |
|---|---|---|
| session_id | Session identifier | Идентификатор сеанса наблюдений |
| mode_id | Observation mode identifier | Номер режима наблюдений внутри сеанса |
| Nerr | Number of errors array | Массив ошибок по блокам NAND |
| Nmax | Max number of events | Максимум событий для формата «События» |
| Nhist | Histogram size | Число каналов гистограммы |
| Phist | Histogram parameter | Параметр конфигурации `Спектр-1` |
| RTC time | Instrument time | Приборное время RTC |
| Board time | Onboard time | Бортовое время КА |
| packet_index | Packet index | Номер текущего пакета |
| block_index | Block index | Номер текущего блока NAND |
| safe-state | Safe state | Безопасная конфигурация |

---

## 13. Термины, которые лучше использовать единообразно

| Предпочтительный термин | Не рекомендуется | Причина |
|---|---|---|
| режим наблюдений | режим сбора | В документации используется “наблюдения” |
| телеметрическое сообщение | ответка / телеметрия наружу | Нужна точность |
| аварийный статус | ошибка | Ошибка и авария не всегда одно и то же |
| команда управления | пакет команды | Команда — логическая сущность |
| команда телеметрии | входная телеметрия | Лучше не путать с ТС |
| пакет НИ | кусок данных | В документации уже есть термин |
| активный банк NAND | текущая флешка | Не скатываться в бытовой жаргон |
| safe-состояние | безопасная конфигурация | Использовать как синонимы, но осознанно |

---

## 14. Правила использования английских идентификаторов

В документации приняты следующие правила:
- для режимов и подсостояний используются короткие английские идентификаторы (`OBSERVE`, `TEST_SAVE`);
- в поясняющем тексте даётся русское наименование;
- для команд используются английские символические имена (`CMD_SHUTDOWN`, `TLM_ORBIT`);
- для аварий используются обозначения из протокола (`ALARM_NAND`, `ALARM_USB`);
- в итоговой LaTeX-версии допускается сохранить английские идентификаторы как моноширинные обозначения.

---

## 15. Открытые вопросы по терминологии

В этом разделе фиксируются термины, которые ещё нужно окончательно стабилизировать:
- использовать ли “пакет НИ” или “пакет научной информации” везде одинаково;
- писать ли “safe-состояние” или только “безопасное состояние”;
- оставить ли английские имена подсостояний в основном тексте или вынести их только в таблицы и диаграммы;
- использовать ли “ППЗУ” и “NAND” как полные синонимы или различать их по контексту.

---

## 16. Связь с другими документами

Документ должен использоваться совместно со всеми algorithm-файлами проекта:
- `Описание_алгоритма_штатного_ПО_НА_NATALIA.md`
- `Таблица_состояний_и_переходов_NATALIA.md`
- `Обработка_команд_CAN_NATALIA.md`
- `Аварии_и_защитная_логика_NATALIA.md`
- `modes/*.md`
- `diagrams/*.mmd`