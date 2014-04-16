## SQL snippets (or more beautiful description)

### 83compat.sql

### check_are_all_subscribed.sql

### check_missing_grants.sql

### check_strange_fk.sql

### check_uniq_indexes.sql
Searhes the tables which do not have UNIQUE CONSTRAINTs.

### create_db_activity_view9.2.sql and create_db_activity_view.sql
Creating VIEWs for viewing running postgres processes with transactions in which the runtime is more than 100ms, and queries with runtime more than 500ms. Requires enabled track_activities in postgresql.conf for proper display the state of processes. This views allows to view only running processes, cutting idle processes. create_db_activity_view.sql is used in PostgreSQL version 9.1 and older, create_db_activity_view9.2.sql used since PostgreSQL 9.2.

Fields:

* ts_age - transaction runtime

* state - process state (active, idle, idle in transaction, idle in transaction (aborted), fastpath function call or disabled when track_activities off)

* query_age - current query runtime;

* change_age - time elapsed since the last change of process state;

* datname - database name in which process is connected;

* pid - postgres process id;

* usename - database username which used in postgres process for running query;

* waiting - set in true if current process waiting other query (sometimes not good);

* client_addr - remote client ip address which connected to postgres;

* client_port - remote client port number which is used in connection;

* query - query text which is currently executed in process;

### create_query_stat_cpu_time_view.sql

Создание VIEW query_stat_cpu_time которая отображает информацию о запросах (включая текст запросов) с временем выполнения >= 0.02 секунд (без учета времени выполнения IO). Требует наличия  pg_stat_statements и опционально включенного  track_io_timings. (Описание полей ниже в create_query_stat_time_view.sql)

### create_query_stat_io_time_view.sql

Создание VIEW query_stat_io_time которая отображает информацию о запросах (включая текст запросов) с временем выполнения блочного ввода-вывод >= 0.02 секунд. Требует наличия  pg_stat_statements и включения  track_io_timings. (Описание полей ниже в create_query_stat_time_view.sql)

### create_query_stat_log.sql

### create_query_stat_time_view.sql

Создание VIEW query_stat_time которая отображает информацию о запросах (включая текст запросов) с общим временем >= 0.02 секунд (в т.ч. и блочного IO). Требует наличия  pg_stat_statements и опционального включения  track_io_timings.

Колонки:

* time_percent - общее время выполнения запроса в %, на фоне времени выполнения всех запросов

* iotime_percent - время выполнения блочного ввода-вывода в %, на фоне общего для всех запросов времени выполнения блочного IO

* cputime_percent - время выполнения (без учета блочного IO) в %, на фоне общего для всех запросов времени (без учета выполнения блочного IO)

* total_time - общее время выполнения всех вызовов данного запроса

* avg_time - среднее время выполнения для запроса

* avg_io_time - среднее время выполнения блочного IO для запроса

* calls - количество вызовов текущего запроса

* calls_percent - количество вызовов текущего запроса в %, от общего числа вызовов всех запросов

* rows - количество строк возвращенное в результате выполнения текущего запроса

* row_percent - количество строк возвращенное в результате выполнения текущего запроса в %, на фоне числа строк возвращенными всеми запросами

* query - текст запроса

Замечание: все запросы, время выполнения которых меньше 0.02 секунд, суммируются в отдельную строку с query = other.

Применение:

### create_slonik_set_full.sql

### create slonik_set_incremental.sql

### create_xlog_math_procedures.sql

Создание трех функций:

* xlog_location_numeric - показывает текущую позицию записи WAL журнала в десятичном выражении.

* replay_lag_mb - показывает приблизительное отставание по записи WAL в мегабайтах.

* all_replayed - возвращает true если воспроизведены все WAL (т.е. лаг отсутствует)

Применение:

### db_activity.sql and db_activity9.2.sql

### dirty_to_read_stat.sql

Статистика чтения "dirty" данных. Требует наличия  pg_buffercache

Колонки: relation - имя объекта (+схема)

* relkind - тип объекта из r = ordinary table, i = index, S = sequence, v = view, c = composite type, t = TOAST table

* tblsp - tablespace

* relsize - размер таблицы, в MB

* %_cached - приблизительный объем страниц, в кэше в %

* disk_read - объем прочитанных данных с диска (разница значений из функций pg_stat_get_blocks_fetched и pg_stat_get_blocks_hit)

* dirty_size - объем измененных страниц, в Mb

* buffer_data.%_dirty - объем измененных страниц, в %

* read_to_dirty_ratio - оношение объема прочитанных данных с диска к количеству "грязных" страниц, т.е. количество чтения на одну грязную страницу в среднем

Применение:

### generate_drop_items.sql

### index_candidates_from_ssd.sql и index_candidates_to_ssd.sql

Отображает индексы которые следует вынести с SSD или наоборот перенести на SSD

Колонки:

* table - таблица

* index - индекс

* tblsp - tablespace

* size - размер таблицы (pretty)

* disk - объем прочитанных данных с диска по указанному индексу, в блоках

* disk_rat - объем диского чтения, в %

* d_w_rat - отношение числа блоков прочитанных с диска к суммарному изменению строк в таблице (insert+update+delete-hot_update)

* write - суммарное число записи (в tuples) по таблице (insert+update+delete-hot_update)

* idx_scan - количество чтений индекса

* idx_tup_read - количество строк взятых из индекса

Маленькое значение d_w_rat говорит о маленьком чтении с диска при относительно больших изменениях в таблице (которые в свою очередь влияют на перестроение индекса). Для отображения индексов которые рекомендуется вынести c SSD, в sql есть условие: отображать индексы с d_w_rat < 25 и tblsp = "ssd"
Большое значение d_w_rat говорит о большом чтении с диска (плохо) при относительно небольших изменениях в таблице.
Для отображения индексов которые рекомендуется разместить на SSD, в sql есть условие: отображать индексы с d_w_rat > 10, disk > 1000 и tblsp != "ssd"

### index_disk_activity.sql

Отображает дисковое чтение по индексам

Колонки:

* table - таблица (+схема) которой принадлежит индекс

* index - имя индекса

* tblsp - tablespace

* size - размер индекса (pretty)

* disk - объем прочитанных данных с диска по указанному индексу, в блоках

* disk_rat - объем дискового чтения, в %

* d_w_rat - отношение числа блоков прочитанных с диска к суммарному изменению строк в таблице (insert+update+delete-hot_update)

* write - суммарное число записи (в tuples) по таблице (insert+update+delete-hot_update)

* idx_scan - количество чтений индекса

* buffer_data.dirty - объем "грязных" данных (pretty)

* buffer_data.%_dirty - объем "грязных" данных, в %

Условие выборки disk > 100

### indexes_with_null.sql

Отображает список индексов в которых большое количество NULL

Колонки:

* table - имя таблицы

* index - имя индекса

* field - колонка

* statnullfrac - доля NULL в колонке

* indexsize - размер индекса

* indexdef - описание индекса

условия выборки: statnullfrac > 0.5 и pg_relation_size > 81920 bytes.

### low_used_indexes.sql

Показывает редкоиспользуемые индексы - это те в которых мало индекс-сканов и много записи в таблицы к которым они принадлежат

Колонки:

* schemaname.relname - схема и таблица которой принадлежит индекс

* indexrelname - имя индекса

* idx_scan - количество чтений индекса

* write_activity - суммарная запись в таблицу которой принадлежит индекс (INSERT/UPDATE/DELETE)

* seq_scan - число последовательных чтений таблицы

* n_live_tup - число живых строк

* size - размер индекса

условия: (idx_scan / write_activity) < 0.01 и write_activity > 10000

### master_wal_position.sql

### query_stat_counts.sql

Показывает статистику по запросам: запросы, количество вызовов, процент выполнения по времени на общем фоне, средние величины.

Колонки:

* time_percent - время выполнения запросов от общего времени выполнения всех запросов, в %

* total_time - общее время выполнения запроса

* avg_time - среднее время выполнения запроса, в секундах

* calls - количество вызовов запроса

* calls_percent - количество вызовов запроса на фоне общего количества всех запросов, в %

* rows - количество строк возвращенных запросом

* row_percent - количество строк на фоне общего стро возвращенных всеми запросов, в %

* query - текст запроса

условие: (calls / sum(calls)) >= 0.01, все что меньше, отображается в особой строке c query = 'other'.

### query_stat_cpu_time.sql, query_stat_io_time.sql, query_stat_rows.sql, query_stat_time.sql 

Запросы аналогичные тем что создаются во VIEW query_stat_cpu_time, query_stat_io_time, query_stat_time и показывают время выполнения запросов с учетом как процессорного и блочного IO(query_stat_time.sql), так и в отдельном виде (query_stat_io_time.sql и query_stat_cpu_time.sql). Требует наличия  pg_stat_statements и опционально включенного  track_io_timings.

### redundant_indexes.sql

Показывает наличие избыточных индексов, избыточный индекс определяется как индекс поля, который может являться частью другого индекса.

Колонки:

* main_index - основной индекс (SQL)

* redundant_index - избыточный индекс (SQL)

* redundant_index_size - размер избыточного индекса (pretty)

### seq_scan_tables.sql

Отображает таблицы которые читаются последовательно (в т.ч. и при наличии индексов).

Колонки:

* schemaname.relname - искомая таблица

* n_live_tup - приблизительное количество "живых" строк

* seq_scan - количество последовательных проходов

* seq_tup_read - количество строк которые были возвращены через последовательное чтение

* write_activity - объем записи в таблицу (INSERT/UPDATE/DELETE)

* index_count - количество индексов на таблице

* idx_scan - количество проходов по индексам таблицы

* idx_tup_fetch - количество строк которые были возвращены через индексный проход.

Условия: seq_scan > 0 и seq_tup_read > 100000

### set_default_grants.sql

Установка DEFAULT PRIVILEGES для новых создаваемых объектов от имени postgres для роле role_ro и role_rw:

role_ro: select on sequences; select on tables.

role_rw: select,usage on sequences; select,insert,update,delete on tables.

### set_missing_grants.sql

Установка соответствующих GRANT для ролей role_ro (SELECT), role_rw (SELECT,INSERT,UPDATE,DELETE,USAGE) на таблицы, представления и последовательности в случае если ACL этих объектов пуст (NULL) или не соответствует доступу проставляемому настоящим запросом.

### slave_wal_position.sql

Показывает текущее состояние WAL: принятое от мастера и воспроизведенное на данный момент.

### slony_tables.sql

Показывает список таблиц из _slony.sl_table

### sync_tablespaces.sql

Находит индексы которые размещены в других tablespace чем соответсвующие им таблицы и выполняет перенос индексов (ALTER INDEX indexname SET TABLESPACE tablespace) в тот tablespace где размещена соответствующая индексу таблица.

### table_candidates_from_ssd.sql and table_candidates_to_ssd.sql

Показывает таблицы которые следует вытащить с SSD (много записи, мало чтения) или наоборот поместить с SSD (мало записи, много чтения)

Колонки:

* nspname.relnam - таблица

* tblsp - tablespace

* size - размер таблицы, включая TOAST (pretty)

* ratio - доля записи (insert/delete/2*update) на фоне дискового чтения (pg_stat_get_blocks_fetched - pg_stat_get_blocks_hit), с учетом TOAST

* disk - объем дискового чтения (pg_stat_get_blocks_fetched - pg_stat_get_blocks_hit), с учетом TOAST

* disk% - доля дискового чтения связанного с этой таблицей на фоне суммарного чтения всех таблиц (включая TOAST)

* rt_d_rat - отношение извлеченных строк из таблицы и ее индексов к объему дискового чтения связанного с этой таблицей

* r_tuples - количество извлеченных строк из таблицы и ее индексов

* write - количество записи (insert/delete/2*update) включая TOAST, в строках

* write% - доля дискового записи связанной с этой таблицей на фоне суммарной записи во всех таблиц (включая TOAST)

* n_tup_ins - количество вставленных строк, включая TOAST

* n_tup_upd - количество обновленных строк, включая TOAST

* n_tup_del - количество удаленных строк, включая TOAST

условия для вынесения таблиц с SSD: tblsp="ssd" и ratio < 20 (много записи, мало чтения)

условия для размещения таблиц на SSD: tblsp != "ssd" и ratio > 10 (мало записи, много чтения)

### table_disk_activity.sql

Показывает дисковую активность по конкретным таблицам.

Колонки:

* nspname.relname - таблица

* tblsp - tablespace

* size - размер таблицы включая TOAST (pretty)

* ratio - доля записи (insert/delete/2*update) на фоне дискового чтения (pg_stat_get_blocks_fetched - pg_stat_get_blocks_hit), с учетом TOAST

* disk% - доля дискового чтения связанного с этой таблицей на фоне суммарного чтения всех таблиц (включая TOAST)

* rt_d_rat - отношение извлеченных строк из таблицы и ее индексов к объему дискового чтения связанного с этой таблицей

* r_tuples - количество извлеченных строк из таблицы и ее индексов

* write - количество записи (insert/delete/2*update) включая TOAST, в строках

* write% - доля дискового записи связанной с этой таблицей на фоне суммарной записи во всех таблиц (включая TOAST)

* dirty - размер занимаемый грязными страниц (pretty)

* %_dirty - процент грязных страниц от общего числа страниц

условия: pg_stat_get_tuples_fetched > 100 или write > 10

### table_index_write_activity.sql и table_write_activity.sql

Отображает объем записи в индексы таблиц (table_index_write_activity.sql) и объем записи в таблицы (table_write_activity.sql).

Колонки:

* schemaname.relname - таблица

* size - размер таблицы, без учета индексов (pretty)

* tblsp - tablespace

* seq_scan - количество последовательных проходов по таблице

* idx_scan - количество чтений индексов таблицы

* n_tup_ins - количество вставленных строк

* n_tup_upd - количество обновленных строк

* n_tup_del - количество удаленных строк

* total - общая сумма по INSERT/UPDATE/DELETE для table_index_write_activity.sql и сумма по INSERT/2*UPDATE/DELETE для table_write_activity.sql

* hot_rate - доля HOT среди всех обновлений

* fillfactor - значение fillfactor для таблицы

условия для table_index_write_activity.sql total > 100

условия для table_write_activity.sql total > 0