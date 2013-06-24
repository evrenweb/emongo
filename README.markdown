# Emongo
Erlang için MongoDB sürücüsü

#### Emongo'nun amacı, stabil olması ve kullanım açısından çok kolay olmasıdır.

Emongo'nun orjinalini [5HT](https://github.com/5HT/emongo) geliştirmiştir. Bu repo ile Readme'sini türkçeye çevirerek, türk kullanıcılarına hitap, soru ve isteklerini cevaplamak amaçlanmıştır.

## Derleme ve Yükleme

	make
	sudo make install

## emongo'yu başlat

	application:start(emongo).

## mongodb ile bağlantı

#### Seçenek 1 - Ayar dosyası

ornek.config:

	[{emongo, [
		{pools, [
			{pool1, [
				{size, 1},
				{host, "localhost"},
				{port, 27017},
				{database, "testdatabase"}
			]}
		]}
	]}].

erlang ı başlatırken ayar dosyanızın yerini belirtin

	erl -config priv/example

uygulamayı başlatın

	application:start(emongo).

#### Seçenek 2 - Havuz ekleyin

uygulamayı başlatın ve dilediğiniz kadar havuz ekleyin

	application:start(emongo).
	emongo:add_pool(pool1, "localhost", 27017, "testdatabase", 1).

## API Kullanım Referansları

__PoolName__ = atom()  
__Host__ = string()  
__Port__ = integer()  
__Database__ = string()  
__PoolSize__ = integer()  
__CollectionName__ = string()  
__Selector__ = Document  
__Document__ = [{Key, Val}]  
__Key__ = string() | atom() | binary() | integer()  
__Val__ = float() | string() | binary() | Document | {array, [term()]} | {binary, BinSubType, binary()} | {oid, binary()} | {oid, string()} | bool() | now() | datetime() | undefined | {regexp, string(), string()} | integer()  
__BinSubType__ = integer() <http://www.mongodb.org/display/DOCS/BSON#BSON-noteondatabinary>

## Havuz Ekle

	emongo:add_pool(PoolName, Host, Port, Database, PoolSize) -> ok

## Ekle

__PoolName__ = atom()  
__CollectionName__ = string()  
__Document__ = [{Key, Val}]  
__Documents__ = [Document]

	emongo:insert(PoolName, CollectionName, Document) -> ok
	emongo:insert(PoolName, CollectionName, Documents) -> ok

### Örnekler

	%% "collection" koleksiyonuna iki alanlı tek belge eklemek
	emongo:insert(test, "collection", [{"field1", "value1"}, {"field2", "value2"}]).

	%% insert two documents, each with a single field into the "collection" collection
	emongo:insert(test, "collection", [[{"document1_field1", "value1"}], [{"document2_field1", "value1"}]]).

## Güncelle

__PoolName__ = atom()  
__CollectionName__ = string()  
__Selector__ = Document  
__Document__ = [{Key, Val}]  
__Upsert__ = true | false (Eğer var olan belge ile eşleşme olmaz ise yeni belge ekler)
__MultiUpdate__ = true | false (Eğer eşleşen bütün seçilenlerin güncellenmesi gerekiyorsa)

	%% varsayılan upsert == false ve multiupdate == false
	emongo:update(PoolName, CollectionName, Selector, Document) -> ok
	emongo:update(PoolName, CollectionName, Selector, Document, Upsert) -> ok
	emongo:update(PoolName, CollectionName, Selector, Document, Upsert, MultiUpdate) -> ok

### Örnekler

	%% "field1" == "value1" olan belgeyi güncelle
	emongo:update(test, "collection", [{"field1", "value1"}], [{"field1", "value1"}, {"field2", "value2"}]).

## Sil

__PoolName__ = atom()  
__CollectionName__ = string()  
__Selector__ = Document

	%% koleksiyondaki bütün belgeleri sil
	emongo:delete(PoolName, CollectionName) -> ok

	%% seçici ile eşleşen, koleksiyonda bulunan bütün belgeleri sil
	emongo:delete(PoolName, CollectionName, Selector) -> ok

## Bul

__Options__ = {timeout, Timeout} | {limit, Limit} | {offset, Offset} | {orderby, Orderby} | {fields, Fields} | response_options  
__Timeout__ = integer (timeout in milliseconds)  
__Limit__ = integer  
__Offset__ = integer  
__Orderby__ = [{Key, Direction}]  
__Direction__ = 1 (Asc) | -1 (Desc)  
__Fields__ = [Key] = işlem sonucunda listedeki hangi alanların döndüreleceğini belirler
__response_options__ = return #response{header, response_flag, cursor_id, offset, limit, documents}  
__Result__ = [Document] | response()

	emongo:find_all(PoolName, CollectionName) -> Result
	emongo:find_all(PoolName, CollectionName, Selector) -> Result
	emongo:find_all(PoolName, CollectionName, Selector, Options) -> Result

### Örnekler

__limit, offset, timeout, orderby, fields__

	%% 'collection' içindeki dökümanlardan field1'in 1 e eşit olanlarını bul ve eğer sorgu 5 saniyeden fazla sürerse işlemi durdur
	%% sonuçları 100 ile limitle ve başlangıçtan ilk 10 tanesini
	%% sonuç belgelerini field1 alanın artan değerine göre listele
	%% alanları, dönen belgelerin field1 i ile sınırla ( _id alanı her zaman sonuçlara dahildir )
	emongo:find_all(test, "collection", [{"field1", 1}], [{limit, 100}, {offset, 10}, {timeout, 5000}, {orderby, [{"field1", asc}]}, {fields, ["field1"]}]).

__büyüktür, küçüktür, büyük eşit, küçük eşit__

	%% field1 'in 5 den büyük ve 10 dan küçük değerlerini bul
	emongo:find_all(test, "collection", [{"field1", [{gt, 5}, {lt, 10}]}]).

	%% field1 'in büyük eşit 5 ve küçük eşit 10 değerlerini bul
	emongo:find_all(test, "collection", [{"field1", [{gte, 5}, {lte, 10}]}]).

	%% field1 'in 5 den büyük ve 10 dan küçük değerlerini bul
	emongo:find_all(test, "collection", [{"field1", [{'>', 5}, {'<', 10}]}]).

	%% field1 'in büyük eşit 5 ve küçük eşit 10 değerlerini bul
	emongo:find_all(test, "collection", [{"field1", [{'>=', 5}, {'=<', 10}]}]).

__eşit değil__

	%% field1 'in 5 yada 10 a eşit olmayan değerlerini bul
	emongo:find_all(test, "collection", [{"field1", [{ne, 5}, {ne, 10}]}]).

	%% field1 'in  5'e eşit olmayan değerlerini bul
	emongo:find_all(test, "collection", [{"field1", [{'=/=', 5}]}]).

	%% field1 'in  5'e eşit olmayan değerlerini bul
	emongo:find_all(test, "collection", [{"field1", [{'/=', 5}]}]).

__in__

	%% field1 'in  [1,2,3,4,5] değerlerinden birisini içeren belgeleri bul
	emongo:find_all(test, "collection", [{"field1", [{in, [1,2,3,4,5]}]}]).

__not in__

	%% field1 'in [1,2,3,4,5] değerlerinden birini içermeyen belgeleri bul
	emongo:find_all(test, "collection", [{"field1", [{nin, [1,2,3,4,5]}]}]).

__all__

	%% field1 'in [1,2,3,4,5] değerlerinden hepsini içeren belgeleri bul
	emongo:find_all(test, "collection", [{"field1", [{all, [1,2,3,4,5]}]}]).

__size__

	%% field1 'in dizi boyutu 10 olan belgeleri bul
	emongo:find_all(test, "collection", [{"field1", [{size, 10}]}]).

__exists__

	%% field1 olan değerleri bul
	emongo:find_all(test, "collection", [{"field1", [{exists, true}]}]).

__where__

	%% field1 'in 10 dan büyük değerlerini bul
	emongo:find_all(test, "collection", [{where, "this.field1 > 10"}]).

__nested queries (iç içe sorgular)__

	%% adress alanını alt belge içeren belgelerden
	%% street değeri "Maple Drive" olanını bul
	%% örn: [{"address", [{"street", "Maple Drive"}, {"zip", 94114}]
	emongo:find_all(test, "people", [{"address.street", "Maple Drive"}]).

## Drop database

	%% var olan database'i sil
	emongo:drop_database(PoolName) -> ok

## Tests

[etap](https://github.com/ngerakines/etap) 'a sahip olduğunuzdan emin olun.

    git clone https://github.com/ngerakines/etap.git
    cd etap && make && cd ..
    export ERL_LIBS="etap"

    make test
