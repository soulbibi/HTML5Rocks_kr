App Cache 초보자 사용 가이드 [원문](http://www.html5rocks.com/en/tutorials/appcache/beginner/#toc-updating-cache)
======
*Eric Bidelman - Developer Relations, Google* <br/>
*June 18.2010*<br>
*번역주: 읽기 좋게 의역이 포함되었습니다.* [원문](http://www.html5rocks.com/en/tutorials/appcache/beginner/#toc-updating-cache)

## Index
* Intorduction
* The cache manifest file
    * Referencing a manifest file
    * Structure of a manifest file
* Updating the cache
    * Cache status
    * AppCache Event
* References

Intorduction
----
> It's becoming increasingly important for web-based applications to be accessible offline. Yes, all browsers have caching mechanisms, but they're unreliable and don't always work as you might expect. HTML5 addresses some of the annoyances of being offline with the ApplicationCache interface. 

웹 기반 어플리케이션을 오프라인 상태에서 사용하는  문제가 갈수록 중요해지고 있다. 물론 모든 브라우저에는 이미 이를 위한 캐싱 메카니즘이 있지만 현재의 캐싱은 신뢰도가 낮고 기대와 다르게 작동할때가 많았다. 그래서 HTML5에는 오프라인 비동기처리 수단의 하나로 [ApplicatonCache](http://www.whatwg.org/specs/web-apps/current-work/#applicationcache)가 포함됐다. 

> Using the cache interface gives your application three advantages: 

캐시 인터페이스를 사용하면 세가지 이점이 있다. 

> * Offline browsing - users can navigate your full site when they're offline
* Speed - cached resources are local, and therefore load faster.
* Reduced server load - the browser will only download resources from the server that have changed.

* 오프라인 브라우징 - 사용자가 오프라인일때도 여러분 사이트를 돌아다닐 수 있다. 
* 속도 - 로컬에 리소스를 캐시하므로 더 빠르다.
* 서버부하 감소 - 브라우저는 변경된 리소스만 서버에서 받아온다. 

> The Application Cache (or AppCache) allows a developer to specify which files the browser should cache and make available to offline users. Your app will load and work correctly, even if the user presses the refresh button while they're offline. 

개발자는 캐싱해야 할 파일들을 Application Cache(AppCache라고도 함.)로 지정해서 오프라인 사용자가 접근하게 할 수 있다. 그러면 사용자가 오프라인에서 새로고침을 눌러도, 여러분의 앱은 정상적으로 로딩되어 작동할 것이다. 

##캐시 메니페스트(manifest) 파일

> The cache manifest file is a simple text file that lists the resources the browser should cache for offline access. 

캐시 메니페스트 파일은 단순한 텍스트 파일이며, 오프라인 접근시 필요한 브라우저 리소스 캐시 목록이 들어있다. 

### 메니페스트 파일 참조하기
> To enable the application cache for an app, include the manifest attribute on the document's html tag: 

웹 앱에서 application cache를 사용할 때는 문서의 html 태그에 *manifest* 속성을 넣는다. 

```html
    <html manifest="example.appcache">
      ...
    </html>
```

> The manifest attribute should be included on every page of your web application that you want cached. The browser does not cache a page if it does not contain the manifest attribute (unless it is explicitly listed in the manifest file itself. This means that any page the user navigates to that include a manifest will be implicitly added to the application cache. Thus, there's no need to list every page in your manifest. 

캐싱하고자 하는 모든 웹 어플리케이션의 페이지에 *manifest* 속성을 넣어야 한다. 
*manifest* 속성이 없는 페이지는 캐싱되지 않는다.(메니페스트 파일에 캐싱할 파일로 그 페이지 파일명이 들어있지 않는 한 말이다.) 이 말은 다시 말해서 사용자가 *manifest* 속성이 지정된 어떤 페이지를 접근하면 그 페이지가 메니페스트 파일 안에 명시적으로 포함되어있지 않아도 application cache에 암묵적으로 추가한다는 말이다. 따라서 캐싱하려는 html 페이지를 메니페스트 파일에 포함시킬 필요는 없다. 

> The manifest attribute can point to an absolute URL or relative path, but an absolute URL must be under the same origin as the web application. A manifest file can have any file extension, but needs to be served with the correct mime-type (see below). 

*manifest* 속성에는 상대주소와 절대 주소 모두 쓸 수 있다. 하지만 절대 주소는 동일 원본 정책(same origin policy) 하에서만 가능하다. 메니페스트 파일의 확장자는 뭐든 가능하지만 서버에서 mime-type 지정이 필요하다. 다음을 보자 .

```html
	<html manifest="http://www.example.com/example.mf">
	  ...
	</html>
```
> A manifest file must be served with the mime-type text/cache-manifest. You may need to add a custom file type to your web server or .htaccess configuration. 

메니페스트 파일의 mime-type은 반드시 *text/cache-manifest*이여야 한다. 그러니 웹서버나 *.htaccess* 에서 커스텀 파일 타입 추가 작업이 필요할 지도 모르겠다. 

> For example, to serve this mime-type in Apache, add this line to your config file:

다음은 이를 위한 아파치 설정 파일이다.

```sh
	AddType text/cache-manifest .appcache
```

> Or, in your app.yaml file in Google App Engine:

Google App Engine을 쓴다면 app.yaml 파일 설정을 다음 처럼 한다.

```sh
	- url: /mystaticdir/(.*\.appcache)
	  static_files: mystaticdir/\1
	  mime_type: text/cache-manifest
	  upload: mystaticdir/(.*\.appcache)
```



### 메니페스트 파일의 구조. 

> A simple manifest may look something like this:

간단한 형태의 메니페스트 파일을 보자. 

```sh
	CACHE MANIFEST
	index.html
	stylesheet.css
	images/logo.png
	scripts/main.js
```

> This example will cache four files on the page that specifies this manifest file.

HTML 페이지에 이 메니페스트 파일을 지정하면 브라우저 접근시 4개의 파일이 캐싱된다.

> There are a couple of things to note:

App Cache관련해서 몇가지 주의사항이 있다. 

> * The CACHE MANIFEST string is the first line and is required.
* Sites are limited to 5MB worth of cached data. However, if you are writing an app for the Chrome Web Store, using the unlimitedStorage removes that restriction.
* If the manifest file or a resource specified in it fails to download, the entire cache update process fails. The browser will keep using the old application cache in the event of failure.

* **CACHE MANIFEST** 문자열이 첫줄에 반드시 있어야 한다. 필수사항이다.
* 사이트 당  5메가의 캐시 데이터의 한계가 있다. 그러나 [Chrome Web Store](http://code.google.com/chrome/apps/docs/developers_guide.html)용 앱을 짜는 거라면, *unlimitedStorage*를 사용하여 용량 제한을 풀 수 있다. 
* 메니페스트 파일 자체, 혹은 그 안에 정의된 리소스 다운로드에 문제가 발생하면, 캐시 업데이트 전체의 실패를 의미한다. 그러면 브래우저는 이전에 캐싱해둔 application cache를 사용하게 될 것이다. 

> Lets take a look at a more complex example:

좀 더 복잡한 예를 보자.

```sh
    CACHE MANIFEST
    # 2010-06-18:v2
    
    # Explicitly cached 'master entries'.
    CACHE:
    /favicon.ico
    index.html
    stylesheet.css
    images/logo.png
    scripts/main.js
    
    # Resources that require the user to be online.
    NETWORK:
    login.php
    /myapi
    http://api.twitter.com
    
    # static.html will be served if main.py is inaccessible
    # offline.jpg will be served in place of all images in images/large/
    # offline.html will be served in place of all other .html files
    FALLBACK:
    /main.py /static.html
    images/large/ images/offline.jpg
    *.html /offline.html
```

> Lines starting with a '#' are comment lines, but can also serve another purpose. An application's cache is only updated when its manifest file changes. So for example, if you edit an image resource or change a javascript function, those changes will not be re-cached. You must modify the manifest file itself to inform the browser to refresh cached files. Creating a comment line with a generated version number, hash of your files, or timestamp is one way to ensure users have the latest version of your software. You can also programmatically update the cache once a new version is ready as discussed in the Updating the cache section. 

'#'로 시작하는 줄은 주석인데 App Cache에서는 다른 목적으로도 쓰인다. App Cache는 메니페스트 파일이 변경될 때만 업데이트 된다. 예를 들어 여러분이 이미지를 수정하거나 자바스크립트 함수를 변경한다고, 캐시가 갱신 되진 않는다. 그러므로 **브라우저가 캐시된 파일을 갱신할 수 있도록 메니페스트 파일을 수정해야한다.** 바로 이때 주석이 사용되는데, 주석에 버전, 파일 해시 값, 타임스템프를 포함시켜서 생성하는 방법으로 사용자가 마지막 버전을 가져가게 한다. 아래 **캐시 업데이트** 절에서 설명할 캐시 갱신 방법을 이용하면, 스크립트로 캐시를 업데이트 할 수도 있다. 

> A manifest can have three distinct sections: CACHE, NETWORK, and FALLBACK.

메니페스트 파일은 CACHE와 NETWORK 그리고 FALLBACK 이렇게 세 섹션으로 나뉜다. 

> CACHE:<br/>
This is the default section for entries. Files listed under this header (or immediately after the CACHE MANIFEST) will be explicitly cached after they're downloaded for the first time.

*CACHE:*<br/>
전체 중 기본 섹션이다. 이 헤더 밑(아니면, CACHEMANIFEST 바로 아래)의 파일 목록은 처음 접속시 다운로드되며 무조건 캐싱된다. 

> NETWORK:<br/>
Files listed under this section are white-listed resources that require a connection to the server. All requests to these resources bypass the cache, even if the user is offline. Wildcards may be used.

NEWWORK:<br/>
이 섹션의 파일 목록은 무조건 서버 연결이 필요한 리소스 리스트다. 사용자가 오프라인이라 할지라도 이 요청 리소스 목록은 캐시는 무시하고 서버에서 가져오려 한다. 와일드카드(*)를 쓸 수 있다. 

> FALLBACK:<br/>
An optional section specifying fallback pages if a resource is inaccessible. The first URI is the resource, the second is the fallback. Both URIs must be relative and from the same origin as the manifest file. Wildcards may be used.

FALLBack:<br/>
리소스에 접근할 없을때 필요한 폴백(fallback, 대안)를 정의하는 섹션이다. 필수 세션은 아니다. 첫 URI는 리소스고 두번째가 폴백이다. 두 URI는 반드시 파일 타입이 동일 종류여야 하고 메니페스트 파일과 동일 원본 정책하에 있어야 한다. 와일드카드(*) 사용 가능하다. 

> Note: These sections can be listed in any order and each section can appear more than one in a single manifest.

**주의**: 이 세션들은 정해진 순서가 없으며 한 메니페스트 파일에 같은 섹션이 여러개 있을 수 있다. 

> The following manifest defines a "catch-all" page (offline.html) that will be displayed when the user tries to access the root of the site while offline. It also declares that all other resources (e.g. those on remote a site) require an internet connection.

다음 메니페스트 파일을 보면 사용자가 오프라인에서 사이트의 루트를 접근하려 할때 출력될 페이지, 즉 캐시로만 실행할 페이지(offline.html)를 정의했다.(10라인) 그리고 이를 제외한 모든 리소스는 인터넷 접속이 필요하다고 선언되어 있다. 

```sh
    CACHE MANIFEST
    # 2010-06-18:v3
    
    # Explicitly cached entries
    index.html
    css/style.css
    
    # offline.html will be displayed if the user is offline
    FALLBACK:
    / /offline.html
    
    # All other resources (e.g. sites) require the user to be online. 
    NETWORK:
    *
    
    # Additional resources to cache
    CACHE:
    images/logo1.png
    images/logo2.png
    images/logo3.png
```

> Note: The HTML file that references your manifest file is automatically cached. There's no need to include it in your manifest, however it is encouraged to do so.

**주의**: 메니페스트 파일을 참조하는 HTML 파일은 자동으로 캐시된다. 그래서 사실 메니페스트 파일에 포함시킬 필요없지만 명시적으로 적어주는게 좋다. 

> Note: HTTP cache headers and the caching restrictions imposed on pages served over SSL are overridden by cache manifests. Thus, pages served over https can be made to work offline.

**주의**: SSL로 페이지를 제공할때 생기는 캐싱 제약과 HTTP 케시 해더는 application cache의 메니페스트 파일이 덥어 버린다. 따라서 https로 제공되는 페이지도 오프라인에서 작동되게 만들 수 있다. 

##캐시 업데이트

> Once an application is offline it remains cached until one of the following happens:

어플리케이션은 다음 중 하나가 일어날 때 까지는 현재 캐시한 것으로 오프라인을 유지한다. 

> * The user clears their browser's data storage for your site.
* The manifest file is modified. Note: updating a file listed in the manifest doesn't mean the browser will re-cache that resource. The manifest file itself must be alternated.
* The app cache is programatically updated.

* 사용자가 브라우저에서 해당 사이트의 데이터 저장소를 비웠을때.
* 메니페스트 파일이 수정되었을때. 앞에서 말했지만 메니페스트에 나열된 파일이 수정되었다고 브라우저가 그 목록을 재 캐싱한다는 뜻은 아니며, 메니페스트 파일 자체가 수정되야 한다.  
* 앱 캐시를 자바스크립트로 업데이트 할때. 

### 캐시의 상태 값 (applicationCache.status)
> The *window.applicationCache* object is your programmatic access the browser's app cache. Its status property is useful for checking the current state of the cache:

브라우저의 앱 캐시를 자바스크립트로 다룰떄는 *window.applicationCache* 객체를 사용한다. 이 객체의 *status* 프로퍼티로 캐시의 현재 상태를 확인할 수 있다. 

```javascript
	var appCache = window.applicationCache;
	
	switch (appCache.status) {
	  case appCache.UNCACHED: // UNCACHED == 0
	    return 'UNCACHED';
	    break;
	  case appCache.IDLE: // IDLE == 1
	    return 'IDLE';
	    break;
	  case appCache.CHECKING: // CHECKING == 2
	    return 'CHECKING';
	    break;
	  case appCache.DOWNLOADING: // DOWNLOADING == 3
	    return 'DOWNLOADING';
	    break;
	  case appCache.UPDATEREADY:  // UPDATEREADY == 4
	    return 'UPDATEREADY';
	    break;
	  case appCache.OBSOLETE: // OBSOLETE == 5
	    return 'OBSOLETE';
	    break;
	  default:
	    return 'UKNOWN CACHE STATUS';
	    break;
	};
```

> To programmatically update the cache, first call applicationCache.update(). This will attempt to update the user's cache (which requires the manifest file to have changed). Finally, when the applicationCache.status is in its UPDATEREADY state, calling applicationCache.swapCache() will swap the old cache for the new one.

케시를 자바스크립트로 업데이트 하기위해는 먼저 *applicationCache.update()* 를 호출한다. 이 메서드는 변경된 메니페스트 파일을 요청해서 사용자 앱 케시 업데이트를 시도한다. *applicationCache.status* 가 *UPDATEREADY* 상태일때, *applicationCache.swapCache()* 를 호출하면 이전 캐시를 새 것으로 바뀐다.

```javascript
	var appCache = window.applicationCache;
	
	appCache.update(); // 사용자 캐시 갱신 시도
	
	...
	
	if (appCache.status == window.applicationCache.UPDATEREADY) {
	  appCache.swapCache();  // 갱신 성공. 새 캐시로 교채.
	}
```

> Note: Using update() and swapCache() like this does not serve the updated resources to users. This flow simply tells the browser to check for a new manifest, download the updated content it specifies, and repopulate the app cache. Thus, it takes two page reloads to server new content to users, one to pull down a new app cache, and another to refresh the page content. 

이처럼 *update()* 와 *swapCache()* 를 사용한다고 해서 업데이트 된 캐시 자원이 사용자에게 바로 보이진 않는다. 지금 이 과정은 단지 브라우저에서 새 메니패스트 파일을 확인해서 변경된 내용을 다운로드하고 앱 캐시를 갱신하는 것 뿐이다. 그러므로, 사용자에게 새 컨텐츠를 출력하려면 페이지 로딩이 두번 필요하다. 앱 캐시를 끌어오기 위해서 한번,  페이지 컨텐츠를 갱신하기위해서 한번이다. 

> The good news: you can avoid this double reload headache. To update users to the newest version of your site, set a listener to monitor the updateready event on page load: 

이 볼썽사나운 로딩과정은 다행이 피할 수 있다. 사용자에게 새 버전의 사이트를 갱신해 주기 위해서 문서의 onload에 *updateready* 이벤트 리스너를 지정한다. 

```javascript
	// 페이지 로드시 새로 캐쉬받아야 하는지 확인.
	window.addEventListener('load', function(e) {
	
	  window.applicationCache.addEventListener('updateready', function(e) {
	    if (window.applicationCache.status == window.applicationCache.UPDATEREADY) {
	      // 브라우저가 새 앱 캐시를 다운받는다. 
	      // 캐시를 교체하고, 따끈따끈한 새 파일을 받기위해 페이지 리로드.
	      window.applicationCache.swapCache();
	      if (confirm('A new version of this site is available. Load it?')) {
	        window.location.reload();
	      }
	    } else {
	      // 메니페스트 파일이 바뀐게 없다. 제공할 새로운게 없음.
	    }
	  }, false);
	
	}, false);
```

### 앱 캐시 관련 이벤트

> As you may expect, additional events are exposed to monitor the cache's state. The browser fires events for things like download progress, updating the app cache, and error conditions. The following snippet sets up event listeners for each type of cache event:  

예상했겠지만 캐시의 상태를 관찰하기 위해 추가된 이벤트들이 있다. 브라우저에서 다운로드 상태 변경, 앱 캐시의 갱신, 그리고 에러 발생 같은 일이 생길때 이벤트가 일어난다. 다음 코드는 각 캐시 이벤트 타입에 대한 이벤트 리스너 지정법이다. 

```javascript
	function handleCacheEvent(e) {
	  //...
	}
	
	function handleCacheError(e) {
	  alert('Error: Cache failed to update!');
	};
	
	// 메니페스트이 리소스가 모두 캐싱된 후 발생.
	appCache.addEventListener('cached', handleCacheEvent, false);
	
	// 메니페스트를 새로 가져와 캐시 갱신을 해야 하는지 확인. 과정상 항상 가장 먼저 발생.
	appCache.addEventListener('checking', handleCacheEvent, false);
	
	// 갱신된 것을 찾았다. 브라우저가 리소스를 다운로드 시작.
	appCache.addEventListener('downloading', handleCacheEvent, false);
	
	// 메니페스트 파일에 대한 응답이 404나 410, 다운로드 실패, 
	// 아니면 다운로드가 진행중인데 메니페스트가 변경됬을때.
	appCache.addEventListener('error', handleCacheError, false);
	
	// 메이페스트의 모두 다운로드 완료후 발생.
	appCache.addEventListener('noupdate', handleCacheEvent, false);
	
	// 메니페스트 파일에 대한 응답이 404나 410 이면 발생한다. 
	// 앱 캐시 안의 내용이 삭제 된다. 
	appCache.addEventListener('obsolete', handleCacheEvent, false);
	
	// 메니페스트 파일에 나열된 각 리소스를 모두 가져오면 발생
	appCache.addEventListener('progress', handleCacheEvent, false);
	
	// 메니페스트의 리소스들이 새로 다시 다운로드 되었을때 발생.
	appCache.addEventListener('updateready', handleCacheEvent, false);
```

> If the manifest file or a resource specified in it fails to download, the entire update fails. The browser will continue to use the old application cache in the event of such a failure.

메니페스트 파일이나 그 안에 정의된 리소스를 다운로드에 실패하면, 전체가 롤백된다. 브라우저는 이전 application cache를 사용하게 될 것이다. 

참조 페이지
------------
* [ApplicationCache](http://www.whatwg.org/specs/web-apps/current-work/#applicationcache) API specification
