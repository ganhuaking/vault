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
				- 這段很難懂，因為只是個建議，也沒有範例。看建議內容推測，最初的問題在於要如何知道 server 可接受的參數或格式？現在常見的做法像是提供 Swagger 文件，或是實作 [HATEOAS](https://openhome.cc/Gossip/Spring/HATEOAS.html) 的概念，讓 API 的回傳成為文件，也有助於解決最初的問題。
- [HTTP 1.0](https://datatracker.ietf.org/doc/html/rfc1945)
	- HTTP 0.9 出來之後，後來各家瀏覽器和伺服器都開始各自開幹，沒有統一標準，於是就會遇到不合規格的瀏覽器和伺服器無法正常運作的問題
	- 這份 RFC 的分類是 Informational，只是記錄各家常見的實作為何而已，還不算是標準，下一版的 HTTP 1.1 才是。
- [HTTP 1.1](https://datatracker.ietf.org/doc/html/rfc2616) ，以下筆記是參考 [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231)
	- 1997 發布第一版，1999 發布了第二版，也是用了 10 多年之久的 HTTP 1.1
	- > The request method token is the primary source of request semantics; it indicates the purpose for which the client has made this request and what is expected by the client as a successful result.
		- Method token 指出了 client 的目的，同時它也能預期 client 能拿得到的結果
		- Method token 目前討論起來，感覺比較像 URI
	- > The request method's semantics might be further specialized by the semantics of some header fields when present in a request (Section 5) if those additional semantics do not conflict with the method.  For example, a client can send conditional request header fields (Section 5.2) to make the requested action conditional on the current state of the target resource ([RFC7232]).
		- 某些 Header 語法只要不跟 Method 衝突，它都有可能會影響行為
		- 像 IF-MATCH 之類的話法，會跟目標資源的狀態有關
	- > HTTP was originally designed to be usable as an interface to distributed object systems.  The request method was envisioned as applying semantics to a target resource in much the same way as invoking a defined method on an identified object would apply semantics.  The method token is case-sensitive because it might be used as a gateway to object-based systems with case-sensitive method names.
		- HTTP 是被設計給分散式物件系統存取的介面，而 method 指的是會應用在資源上的方法。這就跟在某個類別上直接定義 method 並呼叫非常像。
	- > Unlike distributed objects, the standardized request methods in HTTP are not resource-specific, since uniform interfaces provide for better visibility and reuse in network-based systems [REST].  Once defined, a standardized method ought to have the same semantics when applied to any resource, though each resource determines for itself whether those semantics are implemented or allowed.
		- 跟物件不一樣的是，HTTP 的請求方法並不是依賴於資源的特性，因為介面統一在網路系統上會有更好的可見性和重用性（參考 REST）。
		- 一個統一的方法，應用在任何資源應該都具有相同的語義。
	- > All general-purpose servers MUST support the methods GET and HEAD. All other methods are OPTIONAL.
		- 所有 server 必須支援 GET / HEAD 方法，其他都是可選的
	- > Additional methods, outside the scope of this specification, have been standardized for use in HTTP.  All such methods ought to be registered within the "Hypertext Transfer Protocol (HTTP) Method Registry" maintained by IANA, as defined in Section 8.1.
		- 本 RFC 有預定義了 8 種方法，其他的在 [IANA](https://datatracker.ietf.org/doc/html/rfc7231#section-8.1)
	- > The set of methods allowed by a target resource can be listed in an Allow header field (Section 7.4.1).  However, the set of allowed methods can change dynamically.  When a request method is received that is unrecognized or not implemented by an origin server, the origin server SHOULD respond with the 501 (Not Implemented) status code.  When a request method is received that is known by an origin server but not allowed for the target resource, the origin server SHOULD respond with the 405 (Method Not Allowed) status code.
		- 資源允許的操作方法可以列在 `Allow` header 裡
			- [7.4.1](https://datatracker.ietf.org/doc/html/rfc7231#section-7.4.1) 定義當回 405 狀態碼的時候，必須要有 `Allow` header
		- 當收到一個無法識別或服務沒有實現的方法時，要回 501 狀態碼
		- 當收到一個可識別且有實作的方法，但資源不允許的操作時要回 405 狀態碼