# Emongo
Ejabberd için MongoDB sürücüsü

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

	%% insert a single document with two fields into the "collection" collection
	emongo:insert(test, "collection", [{"field1", "value1"}, {"field2", "value2"}]).

	%% insert two documents, each with a single field into the "collection" collection
	emongo:insert(test, "collection", [[{"document1_field1", "value1"}], [{"document2_field1", "value1"}]]).

## Güncelle

__PoolName__ = atom()  
__CollectionName__ = string()  
__Selector__ = Document  
__Document__ = [{Key, Val}]  
__Upsert__ = true | false (insert a new document if the selector does not match an existing document)
__MultiUpdate__ = true | false (if all documents matching selector should be updated)

	%% by default upsert == false and multiupdate == false
	emongo:update(PoolName, CollectionName, Selector, Document) -> ok
	emongo:update(PoolName, CollectionName, Selector, Document, Upsert) -> ok
	emongo:update(PoolName, CollectionName, Selector, Document, Upsert, MultiUpdate) -> ok

### Örnekler

	%% update the document that matches "field1" == "value1"
	emongo:update(test, "collection", [{"field1", "value1"}], [{"field1", "value1"}, {"field2", "value2"}]).

## Sil

__PoolName__ = atom()  
__CollectionName__ = string()  
__Selector__ = Document

	%% delete all documents in a collection
	emongo:delete(PoolName, CollectionName) -> ok

	%% delete all documents in a collection that match a selector
	emongo:delete(PoolName, CollectionName, Selector) -> ok

## Bul

__Options__ = {timeout, Timeout} | {limit, Limit} | {offset, Offset} | {orderby, Orderby} | {fields, Fields} | response_options  
__Timeout__ = integer (timeout in milliseconds)  
__Limit__ = integer  
__Offset__ = integer  
__Orderby__ = [{Key, Direction}]  
__Direction__ = 1 (Asc) | -1 (Desc)  
__Fields__ = [Key] = specifies a list of fields to return in the result set  
__response_options__ = return #response{header, response_flag, cursor_id, offset, limit, documents}  
__Result__ = [Document] | response()

	emongo:find_all(PoolName, CollectionName) -> Result
	emongo:find_all(PoolName, CollectionName, Selector) -> Result
	emongo:find_all(PoolName, CollectionName, Selector, Options) -> Result

### Örnekler

__limit, offset, timeout, orderby, fields__

	%% find documents from 'collection' where field1 equals 1 and abort the query if it takes more than 5 seconds
	%% limit the number of results to 100 and offset the first document 10 documents from the beginning
	%% return documents in ascending order, sorted by the value of field1
	%% limit the fields in the return documents to field1 (the _id field is always included in the results)
	emongo:find_all(test, "collection", [{"field1", 1}], [{limit, 100}, {offset, 10}, {timeout, 5000}, {orderby, [{"field1", asc}]}, {fields, ["field1"]}]).

__great than, less than, great than or equal, less than or equal__

	%% find documents where field1 is greater than 5 and less than 10
	emongo:find_all(test, "collection", [{"field1", [{gt, 5}, {lt, 10}]}]).

	%% find documents where field1 is greater than or equal to 5 and less than or equal to 10
	emongo:find_all(test, "collection", [{"field1", [{gte, 5}, {lte, 10}]}]).

	%% find documents where field1 is greater than 5 and less than 10
	emongo:find_all(test, "collection", [{"field1", [{'>', 5}, {'<', 10}]}]).

	%% find documents where field1 is greater than or equal to 5 and less than or equal to 10
	emongo:find_all(test, "collection", [{"field1", [{'>=', 5}, {'=<', 10}]}]).

__not equal__

	%% find documents where field1 is not equal to 5 or 10
	emongo:find_all(test, "collection", [{"field1", [{ne, 5}, {ne, 10}]}]).

	%% find documents where field1 is not equal to 5
	emongo:find_all(test, "collection", [{"field1", [{'=/=', 5}]}]).

	%% find documents where field1 is not equal to 5
	emongo:find_all(test, "collection", [{"field1", [{'/=', 5}]}]).

__in__

	%% find documents where the value of field1 is one of the values in the list [1,2,3,4,5]
	emongo:find_all(test, "collection", [{"field1", [{in, [1,2,3,4,5]}]}]).

__not in__

	%% find documents where the value of field1 is NOT one of the values in the list [1,2,3,4,5]
	emongo:find_all(test, "collection", [{"field1", [{nin, [1,2,3,4,5]}]}]).

__all__

	%% find documents where the value of field1 is an array and contains all of the values in the list [1,2,3,4,5]
	emongo:find_all(test, "collection", [{"field1", [{all, [1,2,3,4,5]}]}]).

__size__

	%% find documents where the value of field1 is an array of size 10
	emongo:find_all(test, "collection", [{"field1", [{size, 10}]}]).

__exists__

	%% find documents where field1 exists
	emongo:find_all(test, "collection", [{"field1", [{exists, true}]}]).

__where__

	%% find documents where the value of field1 is greater than 10
	emongo:find_all(test, "collection", [{where, "this.field1 > 10"}]).

__nested queries__

	%% find documents with an address field containing a sub-document
	%% with street equal to "Maple Drive".
	%% ie: [{"address", [{"street", "Maple Drive"}, {"zip", 94114}]
	emongo:find_all(test, "people", [{"address.street", "Maple Drive"}]).

## Drop database

	%% drop current database
	emongo:drop_database(PoolName) -> ok

## Tests

[etap](https://github.com/ngerakines/etap) 'a sahip olduğunuzdan emin olun.

    git clone https://github.com/ngerakines/etap.git
    cd etap && make && cd ..
    export ERL_LIBS="etap"

    make test
