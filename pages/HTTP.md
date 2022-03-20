- HTTP/0.9
	- [1991](https://www.w3.org/Protocols/HTTP/AsImplemented.html) 年發布最初的版本
		- > This request consists of the word "GET", a space, the [document address](https://www.w3.org/Addressing/BNF.html#1) , omitting the "http:, host and port parts when they are the coordinates just used to make the connection. (If a gateway is being used, then a full document address may be given specifying a different naming scheme).
			- 這裡提到 request 由下面三個部分組成
				- `GET`
				- ` `（空格）
				- 文件位址（document address），但省略 `http:`、主機、端口（Port）
			- 若有 gateway，有可能會有不同命名的 scheme。
			  collapsed:: true
				- #+BEGIN_NOTE
				  這裡討論後，推測是指 proxy 的架構有可能會是這樣。
				  #+END_NOTE
			- 一個簡單的範例：
			  
			  ```
			  GET /user
			  ```
		- > The document address will consist of a single word (ie no spaces). If any further words are found on the request line, they MUST either be ignored, or else treated according to the [full HTTP spec](https://www.w3.org/Protocols/HTTP/HTTP2.html) .
			- 文件位址是一個不帶空格的字串，超過的部分會被忽略。
	- [1992](https://www.w3.org/Protocols/HTTP/HTTP2.html) 基本 HTTP 的定義
		- [Methods](https://www.w3.org/Protocols/HTTP/Methods.html) 定義
			- GET
			  id:: 6237380d-8a9f-48ea-a0e9-e8e5e4ca2dea
				- > means retrieve whatever data is identified by the URI, so where the URI refers to a data-producing process, or a script which can be run by such a process, it is this data which will be returned, and not the source text of the script or process. Also used for searches .
				- 使用 GET 會取得一份 URI 所標識的資料。URI 會對應到生成資料的進程（process）或類似進程執行的腳本（script），執行完後再返回資料。而不是回傳進程或腳本的原始碼。
				- 舉反例：Apache + PHP 若沒開 mod_php 的話，就會回傳 PHP 的原始碼，這可能就會不符合 GET 的定義。
			- HEAD
				- > is the same as GET but returns only HTTP headers and no document body.
				- 跟 ((6237380d-8a9f-48ea-a0e9-e8e5e4ca2dea)) 一樣，但只回 header，而不回應 body
			- CHECKOUT
			  id:: 6237382e-d2b7-4726-a434-5908a596c984
				- > Similar to GET but locks the object against update by other people. The lock may be broken by a higher authority or on timeout: in this case a future CHECKIN will fail. ( Phase out? )
				- 類似 ((6237380d-8a9f-48ea-a0e9-e8e5e4ca2dea)) ，但會將物件上鎖（lock）以防止其他人更新。上鎖可能會失敗，比方說已經比上鎖，或是逾時（timeout）
			- SHOWMETHOD
				- > Returns a description (perhaps a form) for a given method when applied to the given object. The method name is specified in a For-Method: field. (TBS)
				- LATER 討論後，覺得它跟 `OPTIONS` 很像，等之後回頭來研究
			- PUT
			  id:: 62373841-0c30-4266-9c96-cc8a83bbfbc8
				- > specifies that the data in the body section is to be stored under the supplied URL. The URL must already exist. The new contenst of the document are the data part of the request. POST and REPLY should be used for creating new documents.
				- 指定 body 裡的 data 去儲存到對應的 URL 裡，URL 必須已存在。新的文件內容將會是資料的一部分。而 ((62373858-b731-487c-b34d-06ab145711c1)) 和 `REPLY` 則是建立新文件。
			- DELETE
				- > Requests that the server delete the information corresponding to the given URL. After a successfull DELETE method, the URL becomes invalid for any future methods.
				- 要求伺服器刪除給定 URL 的資訊。刪除後，後續 URL 在操作任何 method 都要是非法的。
			- POST
			  id:: 62373858-b731-487c-b34d-06ab145711c1
				- > Creates a new object linked to the specified object. The message-id field of the new object may be set by the client or else will be given by the server. A URL will be allocated by the server and returned to the client. The new document is the data part of the request. It is considered to be subordinate to the specified object, in the way that a file is subordinate to a directory containing it, or a news article is subordinate to a newsgroup to which it is posted.
				- 建立連結到指定物件的新物件，伺服器會分配新的 URL 並返回。
				- 討論認為，最後一句話比較好懂：「一個文章從屬於群組」，這是連結物件階層上的意義。
			- LINK
			  id:: 62373862-d577-4cc0-9c2f-0529a26e3ee7
				- > Links an existing object to the specified object.
				- 參考 ((62373858-b731-487c-b34d-06ab145711c1))，這是連結已存在的物件。
			- UNLINK
				- > Removes link (or other meta-) information from an object.
				- 參考 ((62373862-d577-4cc0-9c2f-0529a26e3ee7))，這是取消連結。
			- CHECKIN
				- > Similar to PUT, but releases the lock set on the object. Fails if no lock has been set by CHECKOUT. Suggestion : phase out this (rcs-like) model in favor of the PUT (cvs-like, non-locking) model of code management.
				- 與 ((62373841-0c30-4266-9c96-cc8a83bbfbc8)) 類似，但這是指釋放對物件上的鎖。如果先前沒有用 ((6237382e-d2b7-4726-a434-5908a596c984)) 上鎖的話，則會失敗。
				- 這裡提到了 [RCS](https://dywang.csie.cyut.edu.tw/dywang/linuxProgram/node117.html) 與 CVS 兩個關鍵字，指的是版本控制的概念
			- TEXTSEARCH
				- > The object may be queried with a text string. The search form of the GET method is used to query the object.
				- 這裡看不出用途，但推測是一個設定的方法，如
				  
				  ```
				  TEXTSEARCH /user
				  GET /user?miles+chou
				  ```
			- SPACEJUMP
				- > The object will accept a query whose terms are the cooridnates of a point within the object. The method is implemented using GET with a derived URL .
				- 這個東西就太神奇了，網站上查到的資料是，若資料有三維空間概念的話，可以用這個方法去取對應座標的資料。
			- SEARCH
				- > Proposed only. The index (etc) identified by the URL is to be searched for something matching in some sense the enclosed message. How does the client know what message fromats are acceptable to the server? (Suggestion of Fred Williams)
				- 這段也有也難懂，後來是直接看範例來確認