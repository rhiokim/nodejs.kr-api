node(1) -- evented I/O for V8 JavaScript
========================================

## Synopsis

'Hello World' 라고 응답하는 Node로 작성된 웹서버의 예제 :

	var http = require('http');

	http.createServer(function (request, response) {
	  response.writeHead(200, {'Content-Type': 'text/plain'});
	  response.end('Hello World\n');
	}).listen(8124);

	console.log('Server running at http://127.0.0.1:8124/');


이 서버를 실행하려면 위의 코드를 `example.js` 라는 파일로 저장하고, node 명령어를 실행하세요.

	> node example.js
		Server running at http://127.0.0.1:8124/

이 문서의 모든 예제는 유사한 방식으로 실행이 가능합니다.



## Standard Modules

Node 에는 프로세스가 컴파일되는 일부 모듈이 포함되어 있고, 대부분은 다음에 문서화가 되어
있다. 이 모듈을 사용하는 가장 일반적인 방법은 require('name')의 반환값을 모듈과 같은
이름으로 지역 변수에 할당하는 것이다.

Example:

	var sys = require('sys');

다른 모듈에서 Node를 확장할 수 있다. `'Modules'`를 참조하세요.


## Buffer(버퍼)

순수한 자바스크립트는 Unicode와 궁합이 잘 맞지만 이진 데이터는 그렇지 않다.
TCP 스트림이나 파일 시스템을 처리 해야할 경우 바이트(octet) 스트림으로 처리해야 한다.
Node는 바이트(octet) 스트림 조작, 생성, 사용하기 위해 몇가지 전략을 가지고 있다.

원시 데이터는 `Buffer` 클래스의 인스턴스에 저장된다. `Buffer`는 정수 배열과 비슷하지만
V8 힙 외부에 할당된 원시 메모리를 지원한다. `Buffer`의 크기는 변경될 수 없다.

`Buffer`는 전역 객체이다.

버퍼와 자바스크립트 문자열 개체간의 변환은 명시적인 인코딩 방식을 지정해야 한다. 여기에는 각각 다른 문자열
인코딩이 있다.

* `'ascii'` - 7bit의 ASCII 데이터 전용이다. 이 인코딩 방식은 매우 빠르며, 만약 상위 비트가 설정
되어 있는 경우 제거한다.

* `'utf8'` - Unicode 문자. 많은 웹 페이지와 기타 문서 포맷은 UTF-8을 사용한다.

* `'base64'` - Base64 문자열 인코딩.

* `'binary'` - 원시 이진 데이터를 각 문자의 첫번째 8bit로 사용되는 인코딩 방식. 이 인
코딩 방식은 더 이상 사용 가치가 없고 `Buffer`에서는 되도록 사용하지 말아야 한다.
이 인코딩 방식은 향후 버전에서 제거 될 예정이다.


### new Buffer(size)

Allocates a new buffer of `size` octets.
`size` 바이트의 새로운 버퍼를 할당

### new Buffer(array)

Allocates a new buffer using an `array` of octets.
바이트 `array`를 사용하는 새로운 버퍼를 할당

### new Buffer(str, encoding='utf8')

Allocates a new buffer containing the given `str`.
주어진 `str`을 포함하는 새로운 버퍼를 할당

### buffer.write(string, offset=0, encoding='utf8')

Writes `string` to the buffer at `offset` using the given encoding. Returns
number of octets written.  If `buffer` did not contain enough space to fit
the entire string it will write a partial amount of the string. In the case
of `'utf8'` encoding, the method will not write partial characters.
주어진 인코딩을 사용하여 `string` 버퍼를 `offset` 위치에 기록한다.  기록된 바이트를
반환한다.  만약 `buffer`가 전체 문자열을 삽입하는데 충분한 공간을 갖지 않는 경우, 문
자열의 일부만 쓴다. `'utf8'` 인코딩인 경우 이 메서드는 어떤 문자열도 쓸 수 없다.

Example: write a utf8 string into a buffer, then print it
예: utf8 문자열을 버퍼에 쓰고 그것을 출력한다.

    buf = new Buffer(256);
    len = buf.write('\u00bd + \u00bc = \u00be', 0);
    console.log(len + " bytes: " + buf.toString('utf8', 0, len));

    // 12 bytes: ½ + ¼ = ¾
    

### buffer.toString(encoding, start=0, end=buffer.length)

`encoding`으로 인코딩된 버퍼 데이터를 `start`에서 `end`까지 디코딩 된 문자열로
반환한다.

`buffer.write()` 예시를 참조하세요.


### buffer[index]

Get and set the octet at `index`. The values refer to individual bytes,
so the legal range is between `0x00` and `0xFF` hex or `0` and `255`.
`index` 위치에 있는 바이트를 가져오거나 설정한다.  이 값은 각 바이트를 나타내기 때
문에 정해진 범위는 16진수 `0x00`에서 `0xFF` 또는 0에서 255 사이 이다.

Example: copy an ASCII string into a buffer, one byte at a time:
예: ASCII 문자열을 1 바이트씩 버퍼로 복사한다.

    str = "node.js";
    buf = new Buffer(str.length);

    for (var i = 0; i < str.length ; i++) {
      buf[i] = str.charCodeAt(i);
    }

    console.log(buf);

    // node.js


### Buffer.byteLength(string, encoding='utf8')

Gives the actual byte length of a string.  This is not the same as 
`String.prototype.length` since that returns the number of *characters* in a
string.
문자열 실제 바이트 수를 반환한다. 이것은 문자열의 *문자수*를 반환하나
`String.prototype.length`와는 동일하지 않다.

Example:
예:

    str = '\u00bd + \u00bc = \u00be';

    console.log(str + ": " + str.length + " characters, " +
      Buffer.byteLength(str, 'utf8') + " bytes");

    // ½ + ¼ = ¾: 9 characters, 12 bytes


### buffer.length

The size of the buffer in bytes.  Note that this is not necessarily the size
of the contents. `length` refers to the amount of memory allocated for the 
buffer object.  It does not change when the contents of the buffer are changed.
버퍼의 사이즈의 바이트 값.  이것은 실제적인 내용의 크기가 아니다는 것을 주의하세요. `length`
버퍼 개체에 할당된 메모리 전체를 참조한다.

    buf = new Buffer(1234);

    console.log(buf.length);
    buf.write("some string", "ascii", 0);
    console.log(buf.length);

    // 1234
    // 1234

### buffer.copy(targetBuffer, targetStart, sourceStart, sourceEnd=buffer.length)

Does a memcpy() between buffers.
버퍼들 사이에서 memcpy()를 한다.

Example: build two Buffers, then copy `buf1` from byte 16 through byte 19
into `buf2`, starting at the 8th byte in `buf2`.
예: 버퍼를 2개 만들고 `buf1`의 16번째 바이트부터 19번째 바이트를 `buf2` 8번째 바이트
위치에 복사한다.

    buf1 = new Buffer(26);
    buf2 = new Buffer(26);
  
    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
      buf2[i] = 33; // ASCII !
    }

    buf1.copy(buf2, 8, 16, 20);
    console.log(buf2.toString('ascii', 0, 25));

    // !!!!!!!!qrst!!!!!!!!!!!!!
    

### buffer.slice(start, end)

Returns a new buffer which references the
same memory as the old, but offset and cropped by the `start` and `end`
indexes.
원본 버퍼와 같은 메모리를 참조하지만 기준점 `start`와 `end` 위치까지 잘라내어
새로운 버퍼를 반환한다.

**Modifying the new buffer slice will modify memory in the original buffer!**
**새로운 버퍼를 잘라내는 것은 원래의 버퍼 메모리를 변경할 수 있다.

Example: build a Buffer with the ASCII alphabet, take a slice, then modify one byte
from the original Buffer.
예: ASCII 알파벳 버퍼를 생성하고, 잘라내어 원래 버퍼에서 1바이트를 변경시킨다.

    var buf1 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
    }

    var buf2 = buf1.slice(0, 3);
    console.log(buf2.toString('ascii', 0, buf2.length));
    buf1[0] = 33;
    console.log(buf2.toString('ascii', 0, buf2.length));

    // abc
    // !bc


## EventEmitter

Many objects in Node emit events: a TCP server emits an event each time
there is a stream, a child process emits an event when it exits. All
objects which emit events are instances of `events.EventEmitter`.
Node의 많은 객체들은 이벤트를 생성한다. : TCP 서버는 스트림이 있을 때마다
이벤트를 생성한다.  자식 프로세스는 종료될 때 이벤트를 생성한다.  이벤트를 생성하는
모든 객체는 events.EventEmitter의 인스턴스이다.

Events are represented by a camel-cased string. Here are some examples:
`'stream'`, `'data'`, `'messageBegin'`.
이벤트는 카멜표기법(CamelCase)의 문자열로 표현됩니다. 몇몇 예제: `'stream'`,
`'data'`, `'messageBegin'`

Functions can be then be attached to objects, to be executed when an event
is emitted. These functions are called _listeners_.
함수는 객체에 연결될 수 있고 그것은 이벤트가 생성되었을 때 실행된다.  이 함수는
_listeners_ 라고 부른다.

`require('events').EventEmitter` to access the `EventEmitter` class.
require('events').EventEmitter 에서 eventEmitter 클래스를 액세스 합니다.

All EventEmitters emit the event `'newListener'` when new listeners are
added.
모든 EventEmitters는 새로운 리스너(listenders)가 추가될 때 `'newListener'`
이벤트를 생성합니다.

When an `EventEmitter` experiences an error, the typical action is to emit an
`'error'` event.  Error events are special--if there is no handler for them
they will print a stack trace and exit the program.
EventEmitter에 오류가 발생했을때, 전형적인 문제는 `'error'` 이벤트 생성하는 것이다.
오류 이벤트는 특별하다. - 핸들러가 없는 경우라면 스택 트레이스가 출력되고 프로그램은
종료됩니다.

### Event: 'newListener'

`function (event, listener) { }`

This event is emitted any time someone adds a new listener.
이 이벤트는 누군가(Object)가 새로운 리스너를 추가할 때 언제든지 생성된다.

### Event: 'error'

`function (exception) { }`

If an error was encountered, then this event is emitted. This event is
special - when there are no listeners to receive the error Node will
terminate execution and display the exception's stack trace.
에러가 발생하면 이 이벤트가 생성된다. 이 이벤트는 특별하다. - 에러를 수신할
리스너가 없을때, Node는 실행을 종료하고 예외(exception) 스택 추적을 표시한다.

### emitter.on(event, listener)

Adds a listener to the end of the listeners array for the specified event.
리스너 배열의 끝에 지정된 이벤트를 리스너로 추가한다.

    server.on('stream', function (stream) {
      console.log('someone connected!');
    });


### emitter.removeListener(event, listener)

Remove a listener from the listener array for the specified event.
**Caution**: changes array indices in the listener array behind the listener.
리스너 배열에 지정된 이벤트에 대한 리스너를 제거한다. **주의** : 제거한 리스너 뒤쪽의
리스너 배열의 위치가 변경된다.

    var callback = function(stream) {
	  console.log('someone connected!');
	};
    server.on('stream', callback);
	// ...
	server.removeListener('stream', callback);


### emitter.removeAllListeners(event)

Removes all listeners from the listener array for the specified event.
리스너 배열에서 지정된 이벤트에 대해 모든 리스너를 제거한다.


### emitter.listeners(event)

Returns an array of listeners for the specified event. This array can be
manipulated, e.g. to remove listeners.
지정된 이벤트에 대한 리스너 배열을 반환한다. 이 배열은 변경될 수 있다.
예를 들어 리스너 제거하는 것.

    server.on('stream', function (stream) {
      console.log('someone connected!');
    });
	console.log(sys.inspect(server.listeners('stream'));
	// [ [Function] ]


### emitter.emit(event, [arg1], [arg2], [...])

Execute each of the listeners in order with the supplied arguments.
인수로 전달된 순서대로 리스너를 실행한다.



## Stream

A stream is an abstract interface implemented by various objects in Node.
For example a request to an HTTP server is a stream, as is stdout. Streams
are readable, writable, or both. All streams are instances of `EventEmitter`.
스트림은 Node의 다양한 개체에서 구현되는 추상적인 인터페이스이다. 예를 들어 HTTP 서버
에 대한 요청은 표준 출력과 마찬가지로 스트림이다.  스트림은 읽거나 쓸 수 있고 또는
둘 다 가능하다.  모든 스트림은 `EventEmitter` 의 인스턴스이다.


## Readable Stream

A `Readable Stream` has the following methods, members, and events.
`Readable Stream`은 다음에 오는 메서드, 맴버, 이벤트를 가지고 있다.

### Event: 'data'

`function (data) { }`

The `'data'` event emits either a `Buffer` (by default) or a string if
`setEncoding()` was used.
`'data'` 이벤트는 `Buffer`(기본값) 또는 `setEncoding()` 된 경우에는 문자열
중 하나를 생성한다.

### Event: 'end'

`function () { }`

Emitted when the stream has received an EOF (FIN in TCP terminology).
Indicates that no more `'data'` events will happen. If the stream is also
writable, it may be possible to continue writing.
스트림이 EOF(TCP 용어에서 FIN)를 받을 때 생성된다. `'data'` 이벤트가 더 이상
발생하지 않는 것을 나타낸다. 스트림이 만약 쓸 수도 있다면 쓰는 것은 계속 가능할
것이다.

### Event: 'error'

`function (exception) { }`

Emitted if there was an error receiving data.
데이터 수신 오류가 발생하면 생성된다.

### Event: 'close'

`function () { }`

Emitted when the underlying file descriptor has be closed. Not all streams
will emit this.  (For example, an incoming HTTP request will not emit
`'close'`.)
하부에서 파일 기술자가 닫혔을 때 생성된다. 모든 스트림이 이벤트를 발생하는 것은 아니
다. (예, 들어오는 HTTP 요청은 `'close'` 이벤트를 생성하지 않는다.)

### Event: 'fd'

`function (fd) { }`

Emitted when a file descriptor is received on the stream. Only UNIX streams
support this functionality; all others will simply never emit this event.
스트림에 대한 파일 기술자를 받을 때 생성된다. UNIX 스트림만이 이 기능을 지원한다. 다
른 모든 스트림은 이 이벤트를 생성하지 않는다.

### stream.readable

A boolean that is `true` by default, but turns `false` after an `'error'`
occured, the stream came to an `'end'`, or `destroy()` was called.
기본적으로 true 이며, `'error'`가 발생하면 스트림 `'end'` 에 도달하거나 , 또는
`destory()`가 호출된 후 false 로 설정된 boolean 이다.

### stream.setEncoding(encoding)
Makes the data event emit a string instead of a `Buffer`. `encoding` can be
`'utf8'`, `'ascii'`, or `'base64'`.
데이터 이벤트가 `Buffer` 대신 문자열을 생성하도록 한다. `encoding` 은 `'utf8'`,
`'ascii'` 또는 `'base64'`를 지정할 수 있다.

### stream.pause()

Pauses the incoming `'data'` events.
`'data'` 이벤트의 진입을 멈춘다.

### stream.resume()

Resumes the incoming `'data'` events after a `pause()`.
`pasue()` 후에 `'data'` 이벤트의 진입을 다시 시작합니다.

### stream.destroy()

Closes the underlying file descriptor. Stream will not emit any more events.
하부 파일 기술자를 닫는다.  스트림은 더 이상 이벤트를 생성하지 않을 것이다.



## Writeable Stream

A `Writable Stream` has the following methods, members, and events.
`Writeable Stream` 은 다음에 오는 메서드, 맴버 그리고 이벤트가 있다.

### Event: 'drain'

`function () { }`

Emitted after a `write()` method was called that returned `false` to
indicate that it is safe to write again.


### Event: 'error'

`function (exception) { }`

Emitted on error with the exception `exception`.
`exception` 예외 상황이 발생한다.

### Event: 'close'

`function () { }`

Emitted when the underlying file descriptor has been closed.
하부 파일 기술자가 닫혔을 떄 생성된다.

### stream.writeable

A boolean that is `true` by default, but turns `false` after an `'error'`
occurred or `end()` / `destroy()` was called.
기본적으로 `true`이며 `'error'`가 발생하면 `end()`/`destory()`가 호출된 후
`false`로 설정된 `boolean`입니다.

### stream.write(string, encoding='utf8', [fd])

Writes `string` with the given `encoding` to the stream.  Returns `true` if
the string has been flushed to the kernel buffer.  Returns `false` to
indicate that the kernel buffer is full, and the data will be sent out in
the future. The `'drain'` event will indicate when the kernel buffer is
empty again. The `encoding` defaults to `'utf8'`.
주어진 `encoding`에 `string`을 기록한다.  문자열 커널 버퍼로 플러시되지 않으면 true
를 반환한다.  커널 버퍼가 가득차면 데이터가 ~~

If the optional `fd` parameter is specified, it is interpreted as an integral
file descriptor to be sent over the stream. This is only supported for UNIX
streams, and is silently ignored otherwise. When writing a file descriptor in
this manner, closing the descriptor before the stream drains risks sending an
invalid (closed) FD.
옵션 fd 인수를 지정하면 스트림 전송을 위한 기본 파일 기술자로 해석됩니다.  이것은 UNIX
스트림에서만 지원되며, 나머지에서는 무시된다.  이러한 파일 기술자를 쓰는 경우 ~

### stream.write(buffer)

Same as the above except with a raw buffer.
원시 버퍼를 사용하는 것 이외는 상기와 동일하다.

### stream.end()

Terminates the stream with EOF or FIN.
스트림은 EOF 혹은 FIN와 함께 종료된다.

### stream.end(string, encoding)

Sends `string` with the given `encoding` and terminates the stream with EOF
or FIN. This is useful to reduce the number of packets sent.
주어진 `encoding`으로 `string` 을 보낸 후 스트림을 EOF 또는 FIN 으로 닫는다. 이것은
전송되는 패킷을 줄이는데 유용합니다.

### stream.end(buffer)

Same as above but with a `buffer`.
`buffer` 것을 제외하고는 상기와 동일하다.

### stream.destroy()

Closes the underlying file descriptor. Stream will not emit any more events.
하부 파일 기술자를 닫는다.  스트림은 더 이상 이벤트를 생성하지 않는다.


## Global Object

These object are available in the global scope and can be accessed from anywhere.
여기의 객체는 전역 범위에서 유효하고 어디서나 액세스할 수 있다.

### global

The global namespace object.
글로벌 네임스페이스 객체

### process

The process object. See the `'process object'` section.
프로세스 개체이다.  `'process object'` 섹션을 참조하십시오.

### require()

To require modules. See the `'Modules'` section.
요청한 모듈이다.  `'Modules'` 섹션을 참조하십시오.

### require.paths

An array of search paths for `require()`.  This array can be modified to add custom paths.
`require()` 를 위한 검색 경로의 배열이다.  이 배열은 사용자 지정 경로를 추가하기 위해 변경할 수 있다.

Example: add a new path to the beginning of the search list
예: 검색 목록의 맨 위에 새 경로를 추가하려면

    require.paths.unshift('/usr/local/node');
    console.log(require.paths);
    // /usr/local/node,/Users/mjr/.node_libraries


### __filename

The filename of the script being executed.  This is the absolute path, and not necessarily
the same filename passed in as a command line argument.
실행중인 스크립트의 파일 이름이다. 이것은 절대 경로이며 반드시 명령줄 인수로 전달된 파일이름과 일치하는
것은 아닙니다.


Example: running `node example.js` from `/Users/mjr`
예: 1node example.js` 를 `/Users/mjr`에서 실행

    console.log(__filename);
    // /Users/mjr/example.js

### __dirname

The dirname of the script being executed.
스크립트가 실행되는 디렉토리 이름

Example: running `node example.js` from `/Users/mjr`
예: `node example.js` 를 `/Users/mjr`에서 실

    console.log(__dirname);
    // /Users/mjr


### module(모듈)

A reference to the current module (of type `process.Module`). In particular
`module.exports` is the same as the `exports` object. See `src/process.js`
for more information.행
현재 모듈(타입은 `process.Module`)에 대한 참조이다. 특히 `module.exports`는
`exports`개체와 동일하다.  더 자세한 사항은 src/process.js를 참조하십시오.


## process(프로세스)

The `process` object is a global object and can be accessed from anywhere.
It is an instance of `EventEmitter`.
`process`는 전역객체로 어디서나 액세스할 수 있다.  그것은 `EventEmitter`의 인스
턴스입니다.


### Event: 'exit'

`function () {}`

Emitted when the process is about to exit.  This is a good hook to perform
constant time checks of the module's state (like for unit tests).  The main
event loop will no longer be run after the 'exit' callback finishes, so
timers may not be scheduled.
프로세스를 종료하려고 할 때 생성된다.  이것은 (단위 테스트와 같은)모듈의 상태를 일정
시간 확인하기 위한 적합한 연결이다.  메인 이벤트 루프는 'exit' 콜백이 완료되면 더 이상
작동하지 않기 때문에 타이머가 일정하지 않을 수도 있다.

Example of listening for `exit`:
`exit`를 모니터링하는 예제

    process.on('exit', function () {
      process.nextTick(function () {
       console.log('This will not run');
      });
      console.log('About to exit.');
    });

### Event: 'uncaughtException'

`function (err) { }`

Emitted when an exception bubbles all the way back to the event loop. If a
listener is added for this exception, the default action (which is to print
a stack trace and exit) will not occur.
발생한 예외가 이벤트 루프까지 도착한 경우에 생성된다.  만약 이 예외에 대한 리스너가
추가되는 경우 기본 동작(스택 추적을 출력하고 종료)은 발생하지 않는다.


Example of listening for `uncaughtException`:

    process.on('uncaughtException', function (err) {
      console.log('Caught exception: ' + err);
    });

    setTimeout(function () {
      console.log('This will still run.');
    }, 500);

    // Intentionally cause an exception, but don't catch it.
    nonexistentFunc();
    console.log('This will not run.');

Note that `uncaughtException` is a very crude mechanism for exception
handling.  Using try / catch in your program will give you more control over
your program's flow.  Especially for server programs that are designed to
stay running forever, `uncaughtException` can be a useful safety mechanism.
`uncaughtException` 은 예외 처리를 위해 매우 거친 메커니즘이라는 것을 주의해야 한다.
프로그램에서 try/catch 를 사용하면 프로그램의 흐름을 잘 제어할 수 있다.  특히 서버 프
로그램은 언제까지 계속 실행하도록 설계되므로 `uncaughtException`은 유용하게 안전한
메커니즘이 될 수 있다.


### Signal Events(신호 이벤트)

`function () {}`

Emitted when the processes receives a signal. See sigaction(2) for a list of
standard POSIX signal names such as SIGINT, SIGUSR1, etc.
프로세스가 신호를 받을때 생성된다.  SIGINT, SIGUSR1 다른 POSIX 표준 시그널 이름 목록
은 sigaction(2)를 참조하십시오.

Example of listening for `SIGINT`:
`SIGINT`를 모니터링 하는 예제

    var stdin = process.openStdin();

    process.on('SIGINT', function () {
      console.log('Got SIGINT.  Press Control-D to exit.');
    });

An easy way to send the `SIGINT` signal is with `Control-C` in most terminal
programs.
많은 터미널 프로그램에서 쉽게 `SIGINT`를 보내는 방법은 `Control-C`를 누르는 것입니다.


### process.stdout

A `Writable Stream` to `stdout`.
`stdout` 에 대한 `Writeable Stream` 입니다.

Example: the definition of `console.log`
예: `console.log`의 정의

    console.log = function (d) {
      process.stdout.write(d + '\n');
    };


### process.openStdin()

Opens the standard input stream, returns a `Readable Stream`.의
표준 입력 스트림을 열고 `Readable Stream`을 반환한다.

Example of opening standard input and listening for both events:
표준 입력을 오픈하여 두 이벤트를 모니터링하는 예제

    var stdin = process.openStdin();

    stdin.setEncoding('utf8');

    stdin.on('data', function (chunk) {
      process.stdout.write('data: ' + chunk);
    });

    stdin.on('end', function () {
      process.stdout.write('end');
    });


### process.argv

An array containing the command line arguments.  The first element will be
'node', the second element will be the name of the JavaScript file.  The
next elements will be any additional command line arguments.
명령줄 인수를 포함하는 배열이다.  첫번째 요소는 `node`, 두번째 요소는 JavaScript
파일의 이름이다. 다음 요소들은 수모든 추가적인 명렬줄 인수가 될 것이다.

    // print process.argv
    process.argv.forEach(function (val, index, array) {
      console.log(index + ': ' + val);
    });

This will generate:
이와 같이 출력된다.

    $ node process-2.js one two=three four
    0: node
    1: /Users/mjr/work/node/process-2.js
    2: one
    3: two=three
    4: four


### process.execPath

This is the absolute pathname of the executable that started the process.
프로세스에 의해 시작되어 실행 파일의 절대 경로이다.

Example:
예:

    /usr/local/bin/node


### process.chdir(directory)

Changes the current working directory of the process or throws an exception if that fails.
프로세스의 현재 동작중인 디렉토리를 변경한다. 만약 실패를 하면 예외를 throw 한다.

    console.log('Starting directory: ' + process.cwd());
    try {
      process.chdir('/tmp');
      console.log('New directory: ' + process.cwd());
    }
    catch (err) {
      console.log('chdir: ' + err);
    }


### process.compile(code, filename)

Similar to `eval` except that you can specify a `filename` for better
error reporting and the `code` cannot see the local scope.  The value of `filename`
will be used as a filename if a stack trace is generated by the compiled code.
`eval`과 비슷하지만 더 나은 오류보고를 위해 `filename`를 지정할 수 있고 `code` 로컬 범위에서
불가하다.  `filename` 값은 만약 컴파일된 코드는 추적을 생성할 때 파일 이름으로 사용됩니다.

Example of using `process.compile` and `eval` to run the same code:
`process.compile` 과 `eval` 에서 동일한 코드를 사용하는 예제:

    var localVar = 123,
        compiled, evaled;

    compiled = process.compile('localVar = 1;', 'myfile.js');
    console.log('localVar: ' + localVar + ', compiled: ' + compiled);
    evaled = eval('localVar = 1;');
    console.log('localVar: ' + localVar + ', evaled: ' + evaled);

    // localVar: 123, compiled: 1
    // localVar: 1, evaled: 1

`process.compile` does not have access to the local scope, so `localVar` is unchanged.
`eval` does have access to the local scope, so `localVar` is changed.
`process.compile` 은 로컬 범위에 액세스하지 않기 때문에, `localVar`는 변경되지 않는다.
`eval` 은 로컬 범위에 액세스하기 때문에 `localVar`는 변경됩니다.

In case of syntax error in `code`, `process.compile` exits node.
`code` 에서 문법 오류인 경우는 `process.compile` 는 node를 종료한다.

See also: `Script`
관련 항목: `Script`


### process.cwd()

Returns the current working directory of the process.
프로세스의 현재 동작 디렉토리를 반환한다.

    console.log('Current directory: ' + process.cwd());


### process.env

An object containing the user environment. See environ(7).
사용자 환경을 포함하는 개체이다. environ(7)를 참조하십시오.


### process.exit(code=0)

Ends the process with the specified `code`.  If omitted, exit uses the 
'success' code `0`.
지정된 `code` 프로세스를 종료한다. 만약 생략되면 'success'를 나타내는 `0`으로
종료한다.

To exit with a 'failure' code:
'failure' 코드로 종료하는 예제

    process.exit(1);

The shell that executed node should see the exit code as 1.
node 를 실행하는 쉘 종료 코드가 1인 것을 볼 수 있을 것입니다.


### process.getgid()

Gets the group identity of the process. (See getgid(2).)  This is the numerical group id, not the group name.
프로세스 그룹 ID를 가져온다. (getgid(2) 참조)  이것은 숫자로 된 그룹 아이디로 그룹 이름이 없습니다.

    console.log('Current gid: ' + process.getgid());


### process.setgid(id)

Sets the group identity of the process. (See setgid(2).)  This accepts either a numerical ID or a groupname string.  If a groupname is specified, this method blocks while resolving it to a numerical ID.
프로세스 그룹 아이디를 설정한다. (setgid(2) 참조) 이것은 숫자로 된 아이디와 그룹 이름 문자열 모두를 받아 들인다. 만약 그룹 이름을 지정하면 숫자로 된 아이디를 해결할 수 있을 때까지 이 메서는 차단된다.

    console.log('Current gid: ' + process.getgid());
    try {
      process.setgid(501);
      console.log('New gid: ' + process.getgid());
    }
    catch (err) {
      console.log('Failed to set gid: ' + err);
    }


### process.getuid()

Gets the user identity of the process. (See getuid(2).)  This is the numerical userid, not the username.
프로세스의 사용자 아이디를 가져온다. (getuid(2) 참조) 이것은 숫자의 사용자 아이디, 사용자 이름은 없다.


    console.log('Current uid: ' + process.getuid());


### process.setuid(id)

Sets the user identity of the process. (See setuid(2).)  This accepts either a numerical ID or a username string.  If a username is specified, this method blocks while resolving it to a numerical ID.
프로세스의 사용자 아이디를 설정한다. (setuid(2) 참조).  이것은 숫자로 된 아이디 혹은 사용자 이름 문자열을 모두 받아들인다. 만약 이름을 지정하면 숫자의 아이디를 해결할 수 있을 때까지 이 메서드는 차단된다.


    console.log('Current uid: ' + process.getuid());
    try {
      process.setuid(501);
      console.log('New uid: ' + process.getuid());
    }
    catch (err) {
      console.log('Failed to set uid: ' + err);
    }


### process.version

A compiled-in property that exposes `NODE_VERSION`.
`NODE_VERSION` 을 나타내는 컴파일된 속성입니다.

    console.log('Version: ' + process.version);

### process.installPrefix

A compiled-in property that exposes `NODE_PREFIX`.
`NODE_PREFIX` 를 나타내는 컴파일된 속성입니다.

    console.log('Prefix: ' + process.installPrefix);


### process.kill(pid, signal='SIGINT')

Send a signal to a process. `pid` is the process id and `signal` is the
string describing the signal to send.  Signal names are strings like
'SIGINT' or 'SIGUSR1'.  If omitted, the signal will be 'SIGINT'.
See kill(2) for more information.
프로세스에서 신호를 보낸다. `pid`는 프로세스 아이디, `siginal`은 전송되는 신호를
문자열에 적용한다. 신호 이름은 `SIGINT`나 `SIGUSR1`은 같은 문자열입니다. 생략하
면 신호는 `SIGINT` 입니다. 자세한 내용은 kill(2)를 참조하십시오.

Note that just because the name of this function is `process.kill`, it is
really just a signal sender, like the `kill` system call.  The signal sent
may do something other than kill the target process.



Example of sending a signal to yourself:
자신에게 신호를 전달하는 예제

    process.on('SIGHUP', function () {
      console.log('Got SIGHUP signal.');
    });

    setTimeout(function () {
      console.log('Exiting.');
      process.exit(0);
    }, 100);

    process.kill(process.pid, 'SIGHUP');


### process.pid

The PID of the process.
프로세스의 PID 

    console.log('This process is pid ' + process.pid);

### process.title

Getter/setter to set what is displayed in 'ps'.
'ps' 에서 어떻게 나타나는지 설정하는 getter/setter 이다.


### process.platform

What platform you're running on. `'linux2'`, `'darwin'`, etc.
어떤 플랫폼에서 동작중인지 확인한다. `'linux2'`, `'darwin'`, 기타 등등.

    console.log('This platform is ' + process.platform);


### process.memoryUsage()

Returns an object describing the memory usage of the Node process.
Node 프로세스의 메모리 사용 상황을 설명하는 개체를 반환한다.

    var sys = require('sys');

    console.log(sys.inspect(process.memoryUsage()));

This will generate:
이와 같이 생성합니다.

    { rss: 4935680
    , vsize: 41893888
    , heapTotal: 1826816
    , heapUsed: 650472
    }

`heapTotal` and `heapUsed` refer to V8's memory usage.
`heapTotal` 과 `heapUsed` 는 V8의 메모리 사용률을 참조한다.


### process.nextTick(callback)

On the next loop around the event loop call this callback.
This is *not* a simple alias to `setTimeout(fn, 0)`, it's much more
efficient.
이벤트 루프의 다음 루프에서 콜백을 호출한다. 이것은 setTimeout(fn,0) 간단한
별칭이 아니라 훨씬 효율적이다.

    process.nextTick(function () {
      console.log('nextTick callback');
    });


### process.umask([mask])

Sets or reads the process's file mode creation mask. Child processes inherit
the mask from the parent process. Returns the old mask if `mask` argument is
given, otherwise returns the current mask.
프로세스의 파일 모드 생성 마스크를 설정하거나 가져온다.  자식은 부모 프로세스에서 마스
크를 상속합니다. `mask` 인수가 주어졌다면 원래 마스크를 반환하고 그렇지 않다면 현재
마스크를 반환한다.

    var oldmask, newmask = 0644;

    oldmask = process.umask(newmask);
    console.log('Changed umask from: ' + oldmask.toString(8) +
                ' to ' + newmask.toString(8));



## System

These functions are in the module `'sys'`. Use `require('sys')` to access
them.
이 함수는 모듈 'sys'에 있다.  require('sys')를 사용함으로써 이들에 액세스한다.


### sys.print(string)

Like `console.log()` but without the trailing newline.
`console.log()` 와 비슷하지만 다음에 줄바꿈을 계속한다.

    require('sys').print('String with no newline');


### sys.debug(string)

A synchronous output function. Will block the process and
output `string` immediately to `stderr`.
동기 출력 함수이다. 프로세스를 차단하고 즉시 `string`을 `stderr`에 출력한다.

    require('sys').debug('message on stderr');


### sys.log(string)

Output with timestamp on `stdout`.
타임스탬프와 함께 `stdout`에 출력한다.

    require('sys').log('Timestmaped message.');


### sys.inspect(object, showHidden=false, depth=2)

Return a string representation of `object`, which is useful for debugging.
디버깅에 유용한 `Object` 문자열을 반환한다.

If `showHidden` is `true`, then the object's non-enumerable properties will be
shown too.
`showHidden` 이 `true` 이면 객체의 Enumberable 에 없는 속성도 표시된다.

If `depth` is provided, it tells `inspect` how many times to recurse while
formatting the object. This is useful for inspecting large complicated objects.
`depth`가 주어진 경우, 개체를 포맷화하기 위해서 여러번 반복하는 방법을 inspect에서
하고 있다. 이것은 크고 복잡한 개체를 조사하는데 유용하다.

The default is to only recurse twice.  To make it recurse indefinitely, pass
in `null` for `depth`.
기본적으로 2번 반복한다.  무한 반복하려면 Depth에 null을 전달한다.

Example of inspecting all properties of the `sys` object:
`sys` 개체에 모든 속성을 조사하는 예제

    var sys = require('sys');

    console.log(sys.inspect(sys, true, null));


### sys.pump(readableStream, writeableStream, [callback])

Experimental
실험

Read the data from `readableStream` and send it to the `writableStream`.
When `writeableStream.write(data)` returns `false` `readableStream` will be
paused until the `drain` event occurs on the `writableStream`. `callback` gets
an error as its only argument and is called when `writableStream` is closed or
when an error occurs.
`readableStream` 에서 데이터를 읽고 `writeableStream` 으로 보낸다.
`writableStream.write(data)` 가 `false`를 반환하면 `writableStream` 이 `Drain`
이벤트를 생성할 때까지 `readableStream` 이 중단된다. writableStream이 종료되면
callback이 호출된다.


## Timers

### setTimeout(callback, delay, [arg], [...])

To schedule execution of `callback` after `delay` milliseconds. Returns a
`timeoutId` for possible use with `clearTimeout()`. Optionally, you can
also pass arguments to the callback.
`delay` 밀리초가 경과 후 `callback`이 실행되도록 예약한다. `clearTimeout()`에서 사용할
수 있는 `timeoutId`를 반환한다. 옵션으로 콜백에 인자를 전달할 수 있다.

### clearTimeout(timeoutId)

Prevents a timeout from triggering.
타임아웃이 트리거되는 것을 방지한다. 

### setInterval(callback, delay, [arg], [...])

To schedule the repeated execution of `callback` every `delay` milliseconds.
Returns a `intervalId` for possible use with `clearInterval()`. Optionally,
you can also pass arguments to the callback.
`delay` 밀리초 간격으로 `callback`이 반복 실행하도록 예약한다. `clearInterval()`에서 사용할
수 있는 intervalId를 반환한다. 옵션으로 콜백에 인자를 전달할 수 있다.

### clearInterval(intervalId)

Stops a interval from triggering.
interval 이 트리거되는 것을 방지한다.


## Child Processes

Node provides a tri-directional `popen(3)` facility through the `ChildProcess`
class.
Node는 `ChildProcess`클래스를 통해 3방향 `popen(3)` 기능을 제공한다.

It is possible to stream data through the child's `stdin`, `stdout`, and
`stderr` in a fully non-blocking way.
non-blocking 방식으로 자식 프로세스의 `stdin`, `stdout`, 그리고 `stderr`를 통해
데이터 스트림이 가능하다.

To create a child process use `require('child_process').spawn()`.
자식 프로세스는 `require('child_process').spawn()` 를 이용해 생성한다.

Child processes always have three streams associated with them. `child.stdin`,
`child.stdout`, and `child.stderr`.
자식프로세스는 3개의 스트림 `child.stdin`, `child.stdout` 그리고 `child.stderr`와
항상 연관되어 있다.

`ChildProcess` is an `EventEmitter`.
`ChildProcess` 는 `EventEmitter` 이다.

### Event:  'exit'

`function (code, signal) {}`

This event is emitted after the child process ends. If the process terminated
normally, `code` is the final exit code of the process, otherwise `null`. If
the process terminated due to receipt of a signal, `signal` is the string name
of the signal, otherwise `null`.
이 이벤트는 자식 프로세스가 종료된 후에 생성된다. 프로세스가 정상적으로 종료된 경우,
`code`는 프로세스 종료 코드이다. 아니면 null 이다. 프로세스가 시그널을 받고 종료하면
시그널은 그 시스널의 문자열 이름이다. 그렇지 않으면 null 이다.

After this event is emitted, the `'output'` and `'error'` callbacks will no
longer be made.
See `waitpid(2)`.
이 이벤트가 생성된 이후, `'output'`과 `'error'` 콜백은 더이상 없다.
waitpid(2) 를 참조하세요.


### child.stdin

A `Writable Stream` that represents the child process's `stdin`.
Closing this stream via `end()` often causes the child process to terminate.
자식 프로세스의 `stdin`을 표현하는 `Writable Stream` 이다. 많은 경우, `end()`를 통해 이
스트림을 닫으면 자식 프로세스가 종료되는 원인이 된다.

### child.stdout

A `Readable Stream` that represents the child process's `stdout`.
자식 프로세스의 `stdout`를 표현하는 `Readable Stream` 이다.

### child.stderr

A `Readable Stream` that represents the child process's `stderr`.
자식 프로세스의 `stderr`를 표현하는 `Readable Stream` 이다.

### child.pid

The PID of the child process.
자식 프로세스의 PID.

Example:
예제 :

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    console.log('Spawned child pid: ' + grep.pid);
    grep.stdin.end();


### child_process.spawn(command, args=[], [options])

Launches a new process with the given `command`, with  command line arguments in `args`.
If omitted, `args` defaults to an empty Array.
명령줄 인자 `args`를 주어진 `command`로 새로운 프로세스를 시작한다. `args`를 생략하면 빈 배열이
기본이다.

The third argument is used to specify additional options, which defaults to:
세번째 인자는 추가 옵션을 지정하는데 사용되며, 그 기본값은 :

    { cwd: undefined
    , env: process.env,
    , customFds: [-1, -1, -1]
    }

`cwd` allows you to specify the working directory from which the process is spawned.
Use `env` to specify environment variables that will be visible to the new process.
With `customFds` it is possible to hook up the new process' [stdin, stout, stderr] to
existing streams; `-1` means that a new stream should be created.
`cwd`는 시작된 프로세스의 작업 디렉토리를 지정할 수 있다. `env`는 새로운 프로세스에 보이는 환경 변수를
지정하는데 사용한다. `customFds`는 새로운 프로세스의 [stdin, stout, stderr] 를 기존의 스트림에
연결하는 것을 허용한다. `-1`은 새로운 스트림이 만들어져야 한다는 걸 의미한다.

Example of running `ls -lh /usr`, capturing `stdout`, `stderr`, and the exit code:
`ls -lh /usr` 실행하고 `stdout`, `stderr`, 종료 코드를 캡쳐하는 예제 :

    var sys   = require('sys'),
        spawn = require('child_process').spawn,
        ls    = spawn('ls', ['-lh', '/usr']);

    ls.stdout.on('data', function (data) {
      sys.print('stdout: ' + data);
    });

    ls.stderr.on('data', function (data) {
      sys.print('stderr: ' + data);
    });

    ls.on('exit', function (code) {
      console.log('child process exited with code ' + code);
    });


Example: A very elaborate way to run 'ps ax | grep ssh'
매우 정교한 방법으로 'ps ax | grep ssh' 를 실행 예제 : 


    var sys   = require('sys'),
        spawn = require('child_process').spawn,
        ps    = spawn('ps', ['ax']),
        grep  = spawn('grep', ['ssh']);

    ps.stdout.on('data', function (data) {
      grep.stdin.write(data);
    });

    ps.stderr.on('data', function (data) {
      sys.print('ps stderr: ' + data);
    });

    ps.on('exit', function (code) {
      if (code !== 0) {
        console.log('ps process exited with code ' + code);
      }
      grep.stdin.end();
    });

    grep.stdout.on('data', function (data) {
      sys.print(data);
    });

    grep.stderr.on('data', function (data) {
      sys.print('grep stderr: ' + data);
    });

    grep.on('exit', function (code) {
      if (code !== 0) {
        console.log('grep process exited with code ' + code);
      }
    });


Example of checking for failed exec:
exec 실패를 확인하는 예제:

    var spawn = require('child_process').spawn,
        child = spawn('bad_command');

    child.stderr.on('data', function (data) {
      if (/^execvp\(\)/.test(data.asciiSlice(0,data.length))) {
        console.log('Failed to start child process.');
      }
    });


See also: `child_process.exec()`
`child_process.exec()` 도 참조하세요.

### child_process.exec(command, [options], callback)

High-level way to execute a command as a child process, buffer the
output, and return it all in a callback.
명령을 자식 프로세스로 실행하여 그 결과를 저장하고 콜백으로 전체를 전달하는
높은 수준의 방법이다.

    var sys   = require('sys'),
        exec  = require('child_process').exec,
        child;

    child = exec('cat *.js bad_file | wc -l', 
      function (error, stdout, stderr) {
        sys.print('stdout: ' + stdout);
        sys.print('stderr: ' + stderr);
        if (error !== null) {
          console.log('exec error: ' + error);
        }
    });

The callback gets the arguments `(error, stdout, stderr)`. On success, `error`
will be `null`.  On error, `error` will be an instance of `Error` and `err.code`
will be the exit code of the child process, and `err.signal` will be set to the
signal that terminated the process.
콜백은 인자 `(error, stdout, stderr)`를 얻는다. 성공 시 `error`는 `null`일 것이다.
에러 시에는 `error`는 `Error`의 인스턴스이고, `err.code`는 자식 프로세스의 종료코드 일것이다.
그리고 `err.signal`은 프로세스를 종료시키는 신이다.

There is a second optional argument to specify several options. The default options are
임의의 두번째 인자로 몇가지 옵션을 지정할 수 있다. 옵션의 기본값은:

    { encoding: 'utf8'
    , timeout: 0
    , maxBuffer: 200*1024
    , killSignal: 'SIGKILL'
    , cwd: null
    , env: null
    }

If `timeout` is greater than 0, then it will kill the child process
if it runs longer than `timeout` milliseconds. The child process is killed with
`killSignal` (default: `'SIGKILL'`). `maxBuffer` specifies the largest
amount of data allowed on stdout or stderr - if this value is exceeded then
the child process is killed.
`timeout`이 0보다 크면, 자식 프로세스 실행 시간이 `timeout` 밀리초보다 길때 kill한다. 자식
프로세스는 `KillSignal` 으로 Kill된다. `maxBuffer` 는 stdout이나 stderr의 데이터 최대량을
정의한다. - 이 값을 초과하면 자식 프로세스는 Kill됩니다.


### child.kill(signal='SIGTERM')

Send a signal to the child process. If no argument is given, the process will
be sent `'SIGTERM'`. See `signal(7)` for a list of available signals.
자식 프로세스에 신호를 보낸다. 인수가 없다면, 자식프로세스는 `'SIGTERM'`를 받게 된다.
가능한 신호 목록은 `SIGNAL(7)`을 참조하세.

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    grep.on('exit', function (code, signal) {
      console.log('child process terminated due to receipt of signal '+signal);
    });

    // send SIGHUP to process
    grep.kill('SIGHUP');

Note that while the function is called `kill`, the signal delivered to the child
process may not actually kill it.  `kill` really just sends a signal to a process.
이 함수는 `KILL`이라 불리지만, 실제로 전달되는 신호는 자식 프로세스를 죽이지 않는다.
`kill`은 단지 프로세스에 신호를 보낼뿐이다. 


See `kill(2)`
`kill(2)`를 참조하세요.



## Script

`Script` class compiles and runs JavaScript code. You can access this class with:
`Script`는 JavasScript 코드를 컴파일하고 실행하는 클래스이다. 다음과 같이 접근할 수 있다.

    var Script = process.binding('evals').Script;

New JavaScript code can be compiled and run immediately or compiled, saved, and run later.
새로운 JavaScript 코드는 컴파일되어 즉시 실행되거나 컴파일 및 저장하여 나중에 실행된다.


### Script.runInThisContext(code, [filename])

Similar to `process.compile`.  `Script.runInThisContext` compiles `code` as if it were loaded from `filename`,
runs it and returns the result. Running code does not have access to local scope. `filename` is optional.
`process.compile`과 유사하다. `Script.runInThisContext`는 `code`를 `filename`에서 로드된 것처럼 컴파일하여 실행
결과를 반환한다.  실행되는 코드는 로컬 범위에 액세스할 수 없다. `filename`은 옵션이다.


Example of using `Script.runInThisContext` and `eval` to run the same code:
`Script.runInThisContext`과 `eval`에서 동일한 코드를 실행하는 예제 :

    var localVar = 123,
        usingscript, evaled,
        Script = process.binding('evals').Script;

    usingscript = Script.runInThisContext('localVar = 1;',
      'myfile.js');
    console.log('localVar: ' + localVar + ', usingscript: ' +
      usingscript);
    evaled = eval('localVar = 1;');
    console.log('localVar: ' + localVar + ', evaled: ' +
      evaled);

    // localVar: 123, usingscript: 1
    // localVar: 1, evaled: 1

`Script.runInThisContext` does not have access to the local scope, so `localVar` is unchanged.
`eval` does have access to the local scope, so `localVar` is changed.
`Script.runInThisContext`는 로컬 범위에 액세스하지 않는다. 그래서, `localVar`는 변경되지 않는다.
`eval`은 로컬 범위에 액세스하기 때문에 `localVar`은 변경된다.


In case of syntax error in `code`, `Script.runInThisContext` emits the syntax error to stderr
and throws.an exception.
`code` 문법 오류가 되는 경우는 `Script.runInThisContext`는 표준 오류에 문법 오류를 출력하고,
예외를 throw 한다.


### Script.runInNewContext(code, [sandbox], [filename])

`Script.runInNewContext` compiles `code` to run in `sandbox` as if it were loaded from `filename`,
then runs it and returns the result. Running code does not have access to local scope and
the object `sandbox` will be used as the global object for `code`.
`sandbox` and `filename` are optional.
`Script.runInNewContext`는 `code`를 `filename`에서 로드된 것처럼 컴파일하고 그것을 `sandbox`에서 실행하여
결과를 반환한다. 실행되는 코드는 로컬 범위에 액세스하지 않고, `sandbox`가 `code`에 있어서의 글로벌 개체로
사용됩니다. `sandbox` 및 `filename`은 옵션이다.

Example: compile and execute code that increments a global variable and sets a new one.
These globals are contained in the sandbox.
예 : 전역 변수를 증가시키고 새로운 값을 설정하는 코드를 실행하고 컴파일한다.

    var sys = require('sys'),
        Script = process.binding('evals').Script,
        sandbox = {
          animal: 'cat',
          count: 2
        };

    Script.runInNewContext(
      'count += 1; name = "kitty"', sandbox, 'myfile.js');
    console.log(sys.inspect(sandbox));

    // { animal: 'cat', count: 3, name: 'kitty' }

Note that running untrusted code is a tricky business requiring great care.  To prevent accidental
global variable leakage, `Script.runInNewContext` is quite useful, but safely running untrusted code
requires a separate process.
?믿을 수 없는 코드의 실행은 대단한 주의가 필요한 위험한 일이다. 일시적인 전역 변수 누수를 방지하기 위해선
`Script.runInNewContext`가 유용하지만, 믿을 수 없는 코드는 분리된 프로세스가 요구된다.


In case of syntax error in `code`, `Script.runInThisContext` emits the syntax error to stderr
and throws an exception.
코드에 문법 오류가 있으면, `Script.runInThisContext`는 문법 오류를 출력하고 에외를 throw 한다.


### new Script(code, [filename])

`new Script` compiles `code` as if it were loaded from `filename`,
but does not run it. Instead, it returns a `Script` object representing this compiled code.
This script can be run later many times using methods below.
The returned script is not bound to any global object.
It is bound before each run, just for that run. `filename` is optional.
`new Script`는 `code`를 `filename`에서 로드된 것처럼 컴파일하지만 실행하지는 않는다. 대신에 컴파일된
코드를 표현하는 `Script` 개체를 반환한다. 이 스크립트는 아래의 메서드를 사용하여 나중에 여러번 실행할
수 있다. 반환되는 스크립트는 어떤 전역 개체와도 연관되어 있지 않다. 각각을 실행하기 전에 연결함으로써
그대로 실행된다. `filename`은 옵션이다.

In case of syntax error in `code`, `new Script` emits the syntax error to stderr
and throws an exception.
`code`에 문법 오류가 있는 경우에 `new Script`는 표준 오류에 문법 오류를 출력하고, 예외를 throw한다.


### script.runInThisContext()

Similar to `Script.runInThisContext` (note capital 'S'), but now being a method of a precompiled Script object.
`script.runInThisContext` runs the code of `script` and returns the result.
Running code does not have access to local scope, but does have access to the `global` object
(v8: in actual context).
`Script.runInThisContext` (대문자 'S'에 주의)과 비슷하지만, 컴파일된 Script 객체의 메소드이다. `script.runInThisContent`는
`script` 코드를 실행하고 그 결과를 반환한다. 실행된 코드는 로컬 범위에 액세스하지 않지만, `global` 개체 (v8 : 실제 컨텍스트)에
액세스한다.

Example of using `script.runInThisContext` to compile code once and run it multiple times:

    var Script = process.binding('evals').Script,
        scriptObj, i;
    
    globalVar = 0;

    scriptObj = new Script('globalVar += 1', 'myfile.js');

    for (i = 0; i < 1000 ; i += 1) {
      scriptObj.runInThisContext();
    }

    console.log(globalVar);

    // 1000


### script.runInNewContext([sandbox])

Similar to `Script.runInNewContext` (note capital 'S'), but now being a method of a precompiled Script object.
`script.runInNewContext` runs the code of `script` with `sandbox` as the global object and returns the result.
Running code does not have access to local scope. `sandbox` is optional.
`Script.runInNewContext` (대문자 'S'에 주의)과 비슷하지만, 컴파일된 `Script` 개체의 메서드이다.
`script.runInNewContext`은 `sandbox`가 글로벌 개체인 것처럼 script 코드를 실행하고 그 결과를 반환한다. 실행되는 코드는
로컬 범위에 액세스하지 않는다. `sandbox`는 옵션이다.

Example: compile code that increments a global variable and sets one, then execute this code multiple times.
These globals are contained in the sandbox.
예 : 전역 변수를 증가하여 설정하는 코드를 컴파일하여 이 코드를 여러 번 실행한다. 이 전역 변수는 샌드 박스에 포함되어 있다.

    var sys = require('sys'),
        Script = process.binding('evals').Script,
        scriptObj, i,
        sandbox = {
          animal: 'cat',
          count: 2
        };

    scriptObj = new Script(
        'count += 1; name = "kitty"', 'myfile.js');

    for (i = 0; i < 10 ; i += 1) {
      scriptObj.runInNewContext(sandbox);
    }

    console.log(sys.inspect(sandbox));

    // { animal: 'cat', count: 12, name: 'kitty' }

Note that running untrusted code is a tricky business requiring great care.  To prevent accidental
global variable leakage, `script.runInNewContext` is quite useful, but safely running untrusted code
requires a separate process.
믿을 수 없는 코드의 실행은 대단한 주의가 필요한 위험한 일이다. 일시적인 전역변수 누수를 방지하기 위해선
`script.runInNewContext`가 유용하지만, 믿을 수 없는 코드는 분리된 프로세스가 요구된다.


## File System

File I/O is provided by simple wrappers around standard POSIX functions.  To
use this module do `require('fs')`. All the methods have asynchronous and
synchronous forms.
File I/O는 POSIX 표준 함수에 대한 간단한 래퍼로 제공된다. 이 모듈을 사용하려면
`require('fs')`를 한다. 모든 메소드는 비동기와 동기의 형식이 있다.

The asynchronous form always take a completion callback as its last argument.
The arguments passed to the completion callback depend on the method, but the
first argument is always reserved for an exception. If the operation was
completed successfully, then the first argument will be `null` or `undefined`.
비동기 형식은 항상 마지막 인자로 완료 콜백을 받는다. 인수로 전달된 완료 콜백 메서드에
의존하지만, 최초의 인수는 항상 예외를 위해 예약되어 있다. 작업이 성공적으로 완료되면
첫 번째 인수는 `null` 또는 `undefined` 이다

Here is an example of the asynchronous version:
비동기 버전의 예 :

    var fs = require('fs');

    fs.unlink('/tmp/hello', function (err) {
      if (err) throw err;
      console.log('successfully deleted /tmp/hello');
    });

Here is the synchronous version:
동기화 버전의 예 :

    var fs = require('fs');

    fs.unlinkSync('/tmp/hello')
    console.log('successfully deleted /tmp/hello');

With the asynchronous methods there is no guaranteed ordering. So the
following is prone to error:
비동기 메소드는 순서를 보장하지 않는다. 그래서 다음과 같은 오류가 나기 쉽다.

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      console.log('renamed complete');
    });
    fs.stat('/tmp/world', function (err, stats) {
      if (err) throw err;
      console.log('stats: ' + JSON.stringify(stats));
    });

It could be that `fs.stat` is executed before `fs.rename`.
The correct way to do this is to chain the callbacks.
`fs.stat`는 `fs.rename`보다 먼저 실행되는 수도 있다. 정확한 방법은
콜백을 연결하는 것다.

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      fs.stat('/tmp/world', function (err, stats) {
        if (err) throw err;
        console.log('stats: ' + JSON.stringify(stats));
      });
    });

In busy processes, the programmer is _strongly encouraged_ to use the
asynchronous versions of these calls. The synchronous versions will block
the entire process until they complete--halting all connections.
처리할게 많은 프로세스에서는 프로그래머가 이런 비동기 버전의 호출을 사용하기를 강력히 권한다.
동기화 버전은 그들이 완료될 때까지 프로세스를 차단한다. 모든 연결을 중단한다.

### fs.rename(path1, path2, [callback])

Asynchronous rename(2). No arguments other than a possible exception are given to the completion callback.
비동기 rename (2). 완료 콜백에는 발생가능 한 예외 이외에는 인수가 전달되지 않는다.

### fs.renameSync(path1, path2)

Synchronous rename(2).
동기화 rename (2).

### fs.truncate(fd, len, [callback])

Asynchronous ftruncate(2). No arguments other than a possible exception are given to the completion callback.
비동기 ftruncate (2). 완료 콜백에는 발생가능 한 예외 이외에는 인수가 전달되지 않는다.

### fs.truncateSync(fd, len)

Synchronous ftruncate(2).
동기화 ftruncate (2).

### fs.chmod(path, mode, [callback])

Asynchronous chmod(2). No arguments other than a possible exception are given to the completion callback.
비동기 chmod (2). 완료 콜백에는 발생가능한 예외 이외에는 인수가 전달되지 않는다.

### fs.chmodSync(path, mode)

Synchronous chmod(2).
동기화 chmod (2).
  
### fs.stat(path, [callback])

Asynchronous stat(2). The callback gets two arguments `(err, stats)` where `stats` is a `fs.Stats` object. It looks like this:
비동기 stat (2). 콜백은 2 개의 인수`(err, stats)`를 받는다. `stats`는 `fs.Stats` 개체이다. 다음과 같다.

    { dev: 2049
    , ino: 305352
    , mode: 16877
    , nlink: 12
    , uid: 1000
    , gid: 1000
    , rdev: 0
    , size: 4096
    , blksize: 4096
    , blocks: 8
    , atime: '2009-06-29T11:11:55Z'
    , mtime: '2009-06-29T11:11:40Z'
    , ctime: '2009-06-29T11:11:40Z' 
    }

See the `fs.Stats` section below for more information.
더 자세한 사항은 아래의 `fs.Stats` 절을 참조하세요.

### fs.lstat(path, [callback])

Asynchronous lstat(2). The callback gets two arguments `(err, stats)` where `stats` is a `fs.Stats` object.
비동기 lstat (2). 콜백은 2 개의 인수`(err, stats)`를 받는다. `stats`는 `fs.Stats` 개체이다. 다음과 같다.

### fs.fstat(fd, [callback])

Asynchronous fstat(2). The callback gets two arguments `(err, stats)` where `stats` is a `fs.Stats` object.
비동기 `fstat(2)`. 콜백은 2 개의 인수`(err, stats)`를 받는다. `stats`는 `fs.Stats` 개체이다. 다음과 같다.

### fs.statSync(path)

Synchronous stat(2). Returns an instance of `fs.Stats`.
동기화 stat(2). `fs.Stats`의 인스턴스를 반환한다.

### fs.lstatSync(path)

Synchronous lstat(2). Returns an instance of `fs.Stats`.
동기화 lstat (2). `fs.Stats`의 인스턴스를 반환한다.

### fs.fstatSync(fd)

Synchronous fstat(2). Returns an instance of `fs.Stats`.
동기화 fstat (2). `fs.Stats`의 인스턴스를 반환한다.

### fs.link(srcpath, dstpath, [callback])

Asynchronous link(2). No arguments other than a possible exception are given to the completion callback.
비동기 link (2). 완료 콜백에는 발생가능한 예외 이외에는 인수가 전달되지 않는다.

### fs.linkSync(dstpath, srcpath)

Synchronous link(2).
동기화 link (2).

### fs.symlink(linkdata, path, [callback])

Asynchronous symlink(2). No arguments other than a possible exception are given to the completion callback.
비동기 symlink (2). 완료 콜백에는 발생가능한 예외 이외에는 인수가 전달되지 않는다.

### fs.symlinkSync(linkdata, path)

Synchronous symlink(2).
동기화 symlink (2).

### fs.readlink(path, [callback])

Asynchronous readlink(2). The callback gets two arguments `(err, resolvedPath)`.
비동기 readlink (2). 콜백은 2 개의 인수 `(err, resolvedPath)` 를 받는다.

### fs.readlinkSync(path)

Synchronous readlink(2). Returns the resolved path.
동기화 readlink (2). 수정된 경로를 반환한다.

### fs.realpath(path, [callback])

Asynchronous realpath(2).  The callback gets two arguments `(err, resolvedPath)`.
비동기 realpath (2). 콜백은 2 개의 인수 `(err, resolvedPath)`를 받는다.

### fs.realpathSync(path)

Synchronous realpath(2). Returns the resolved path.
동기화 realpath (2). 수정된 경로를 반환한다.

### fs.unlink(path, [callback])

Asynchronous unlink(2). No arguments other than a possible exception are given to the completion callback.
비동기 unlink (2). 완료 콜백에는 발생가능한 예외 이외에는 인수가 전달되지 않는다.

### fs.unlinkSync(path)

Synchronous unlink(2).
동기화 unlink (2).

### fs.rmdir(path, [callback])

Asynchronous rmdir(2). No arguments other than a possible exception are given to the completion callback.
비동기 rmdir(2). 완료 콜백에는 발생가능한 예외 이외에는 인수가 전달되지 않는다.

### fs.rmdirSync(path)

Synchronous rmdir(2).
동기화 rmdir(2).

### fs.mkdir(path, mode, [callback])

Asynchronous mkdir(2). No arguments other than a possible exception are given to the completion callback.
비동기 mkdir(2). 완료 콜백에는 발생가능한 예외 이외에는 인수가 전달되지 않는다.

### fs.mkdirSync(path, mode)

Synchronous mkdir(2).
동기화 rmdir(2).

### fs.readdir(path, [callback])

Asynchronous readdir(3).  Reads the contents of a directory.
The callback gets two arguments `(err, files)` where `files` is an array of
the names of the files in the directory excluding `'.'` and `'..'`.
비동기 readdir(3). 디렉토리의 내용을 로드한다. 콜백은 2 개의 인수`(err, files)`를 
받는다. `files` 는 `'..'` 과 `'.'` 를 제외한 디렉토리 안의 파일 이름의 배열이다.

### fs.readdirSync(path)

Synchronous readdir(3). Returns an array of filenames excluding `'.'` and
`'..'`.
동기화 readdir(3). `'..'` 과 `'.'` 를 제외한 파일 이름들의 배열을 반환한다.

### fs.close(fd, [callback])

Asynchronous close(2).  No arguments other than a possible exception are given to the completion callback.
비동기 close (2). 완료 콜백에는 발생가능한 예외 이외에는 인수가 전달되지 않는다.

### fs.closeSync(fd)

Synchronous close(2).
동기화 close (2).

### fs.open(path, flags, mode=0666, [callback])

Asynchronous file open. See open(2). Flags can be 'r', 'r+', 'w', 'w+', 'a',
or 'a+'. The callback gets two arguments `(err, fd)`.
비동기 파일 열기. open(2)를 참조하십시오. 플래그는 'r', 'r', 'w', 'w', 'a'또는 'a+'.
콜백은 2 개의 인수`(err, fd)`를 받는다.

### fs.openSync(path, flags, mode=0666)

Synchronous open(2).
동기화 open(2).

### fs.write(fd, buffer, offset, length, position, [callback])

Write `buffer` to the file specified by `fd`.
fd로 지정된 파일에 `buffer`를 쓴다.

`offset` and `length` determine the part of the buffer to be written.
`offset` 및 `length`에서 버퍼의 어느 부분이 기록되는 여부가 결정된다.

`position` refers to the offset from the beginning of the file where this data
should be written. If `position` is `null`, the data will be written at the
current position.
`position`은 데이터가 기록되는 위치를 파일의 시작 부분에서 오프셋을 나타낸다.
`position`이 `null`이면, 데이터는 현재 위치에서 기록된다.

See pwrite(2).
pwrite(2)를 참조.

The callback will be given two arguments `(err, written)` where `written`
specifies how many _bytes_ were written.
콜백은 2 개의 인수`(err, written)`가 주어진다. `written`는 기록된 `bytes`를
나타낸다.

### fs.write(fd, str, position, encoding='utf8', [callback])

Write the entire string `str` using the given `encoding` to the file specified
by `fd`.

`position` refers to the offset from the beginning of the file where this data
should be written. If `position` is `null`, the data will be written at the
current position.
See pwrite(2).

The callback will be given two arguments `(err, written)` where `written`
specifies how many _bytes_ were written.

### fs.writeSync(fd, buffer, offset, length, position)

Synchronous version of buffer-based `fs.write()`. Returns the number of bytes written.
버퍼 기반 `fs.write()` 의 동기화버전. 기록된 바이트수를 반환한다.

### fs.writeSync(fd, str, position, encoding='utf8')

Synchronous version of string-based `fs.write()`. Returns the number of bytes written.
문자열 기반 `fs.write()` 의 동기화 버전. 기록된 바이트 수를 반환한다.

### fs.read(fd, buffer, offset, length, position, [callback])

Read data from the file specified by `fd`.
`fd`로 지정된 파일에서 데이터를 읽는다.

`buffer` is the buffer that the data will be written to.
`buffer`는 데이터를 쓸 버퍼다.

`offset` is offset within the buffer where writing will start.
`offset`은 쓰기를 시작할 버퍼의 오프셋이다.

`length` is an integer specifying the number of bytes to read.
`length`는 읽어들이는 바이트 수를 지정하는 정수이다.

`position` is an integer specifying where to begin reading from in the file.
If `position` is `null`, data will be read from the current file position.
`position`은 파일의 로드를 시작할 위치를 지정하는 정수이다. `position`이 `null`의
경우, 데이터는 현재 위치에서 로드된다.

The callback is given the two arguments, `(err, bytesRead)`.
callback은 두개이 인수`(err, bytesRead)`를 받는다.

### fs.read(fd, length, position, encoding, [callback])

Read data from the file specified by `fd`.

`length` is an integer specifying the number of bytes to read.

`position` is an integer specifying where to begin reading from in the file.
If `position` is `null`, data will be read from the current file position.

`encoding` is the desired encoding of the string of data read in from `fd`.

The callback is given the three arguments, `(err, str, bytesRead)`.

### fs.readSync(fd, buffer, offset, length, position)

Synchronous version of buffer-based `fs.read`. Returns the number of `bytesRead`.
버퍼 기반 `fs.read`의 동기화 버전. `bytesRead` 수를 반환한다

### fs.readSync(fd, length, position, encoding)

Synchronous version of string-based `fs.read`. Returns the number of `bytesRead`.
문자열 기반 `fs.read`의 동기화버전. `bytesRead` 수를 반환한다.

### fs.readFile(filename, [encoding], [callback])

Asynchronously reads the entire contents of a file. Example:
파일의 전체 내용을 비동기적으로 로드한다. 예 :

    fs.readFile('/etc/passwd', function (err, data) {
      if (err) throw err;
      console.log(data);
    });

The callback is passed two arguments `(err, data)`, where `data` is the
contents of the file.
콜백은 2 개의 인수`(err, data)`를 전달한다. `data`는 파일의 내용이다.

If no encoding is specified, then the raw buffer is returned.
인코딩이 지정되지 않으면, 실 버퍼(raw buffer)가 전달 된다.


### fs.readFileSync(filename, [encoding])

Synchronous version of `fs.readFile`. Returns the contents of the `filename`.
`fs.readFile` 의 동기화 버전 파일내용을 반환한다. 

If `encoding` is specified then this function returns a string. Otherwise it
returns a buffer.
`encoding`을 지정하면 문자열을 반환하지만 그렇지 않으면 버퍼를 반환한다. 


### fs.writeFile(filename, data, encoding='utf8', [callback])

Asynchronously writes data to a file. `data` can be a string or a buffer.
파일에 비동기적으로 데이터를 쓴다.  `data` 는 문자열이나 버퍼일 수 있다. 

Example:
예:

    fs.writeFile('message.txt', 'Hello Node', function (err) {
      if (err) throw err;
      console.log('It\'s saved!');
    });

### fs.writeFileSync(filename, data, encoding='utf8')

The synchronous version of `fs.writeFile`.
`fs.writeFile`의 동기화 버전

### fs.watchFile(filename, [options], listener)

Watch for changes on `filename`. The callback `listener` will be called each
time the file changes.
`filename`의 변경을 감시한다. 콜백  `listener`는 파일 변경 때 만다 호출된다.

The second argument is optional. The `options` if provided should be an object
containing two members a boolean, `persistent`, and `interval`, a polling
value in milliseconds. The default is `{persistent: true, interval: 0}`.
두번째 인수를 선택사항이다. 이 options가 주어지면, 객체는 2개의 boolean 멤버,
`persistent`, `interval`(폴링 간격 밀리초)를 가져야 한다. 기본값은
`{persistent: true, interval: 0}` 이다. 

The `listener` gets two arguments the current stat object and the previous
stat object:
`listener`는 현재상태의 객체와 이전 상태의 객체 두개의 인수를 받는다.

    fs.watchFile(f, function (curr, prev) {
      console.log('the current mtime is: ' + curr.mtime);
      console.log('the previous mtime was: ' + prev.mtime);
    });

These stat objects are instances of `fs.Stat`.
상태 객체는 `fs.Stat`의 인스턴스이다.

### fs.unwatchFile(filename)

Stop watching for changes on `filename`.
`filename` 변경에 대한 감시를 멈춘다. 

## fs.Stats

Objects returned from `fs.stat()` and `fs.lstat()` are of this type.
`fs.stat()`와 `fs.lstat()`에서 반환되는 개체는 이 형식이다.

 - `stats.isFile()`
 - `stats.isDirectory()`
 - `stats.isBlockDevice()`
 - `stats.isCharacterDevice()`
 - `stats.isSymbolicLink()` (only valid with  `fs.lstat()`)
 - `stats.isSymbolicLink()` (`fs.lstat()`에서만 유)
 - `stats.isFIFO()`
 - `stats.isSocket()`


## fs.ReadStream

`ReadStream` is a `Readable Stream`.효
`ReadStream`은 `Readable Stream`이다.

### fs.createReadStream(path, [options])

Returns a new ReadStream object (See `Readable Stream`).
새로운 ReadStream 개체를 반환한다.(`Readable Stream`을 참조하세요).

`options` is an object with the following defaults:
`options`은 다음과 같은 기본값을 갖는 객체이다.

    { 'flags': 'r'
    , 'encoding': null
    , 'mode': 0666
    , 'bufferSize': 4 * 1024
    }

`options` can include `start` and `end` values to read a range of bytes from
the file instead of the entire file.  Both `start` and `end` are inclusive and
start at 0.  When used, both the limits must be specified always.
전체 파일을 로드하는 대신 일부 영역을 로드하므로 `options`에 `start`와 `end`가 포함
될 수 있다. `start`와 `end`는 모두 포괄적으로 0부터 시작한다. 사용할 때 언제든지, 둘
다 지정해야 한다. 사용할 때는 언젠든지, 둘다 지정해야만 한다.

An example to read the last 10 bytes of a file which is 100 bytes long:
100바이트의 길이를 가지는 파일의 마지막 10바이트를 읽는 예

    fs.createReadStream('sample.txt', {start: 90, end: 99});


## fs.WriteStream

`WriteStream` is a `Writable Stream`.
`WriteStream`은 `Writeable Stream` 이다.

### Event: 'open'

`function (fd) { }`

 `fd` is the file descriptor used by the WriteStream.
 `fd`는 WriteStream 에서 사용되는 파일 기술자이다.

### fs.createWriteStream(path, [options])

Returns a new WriteStream object (See `Writable Stream`).
새로운 WriteStream 객체를 반환한다.(`Writable Stream`을 참고하세요).

`options` is an object with the following defaults:
`options`는 다음과 같은 기본값을 갖는 객체이다.

    { 'flags': 'w'
    , 'encoding': null
    , 'mode': 0666
    }


## HTTP

To use the HTTP server and client one must `require('http')`.
HTTP 서버와 클라이언트를 사용하려면 모두 `require('http')`가 필요하다.

The HTTP interfaces in Node are designed to support many features
of the protocol which have been traditionally difficult to use.
In particular, large, possibly chunk-encoded, messages. The interface is
careful to never buffer entire requests or responses--the
user is able to stream data.
Node의 HTTP 인터페이스가 전통적으로 처리가 어려웠던 프로토콜의 많은 기능을 지원하
도록 설게되었다.

HTTP message headers are represented by an object like this:
HTTP 메세지 헤더는 이러한 개체로 표현됩니다.

    { 'content-length': '123'
    , 'content-type': 'text/plain'
    , 'stream': 'keep-alive'
    , 'accept': '*/*'
    }

Keys are lowercased. Values are not modified.
키는 소문자화 되어있다. 값은 변경되지 않는다.

In order to support the full spectrum of possible HTTP applications, Node's
HTTP API is very low-level. It deals with stream handling and message
parsing only. It parses a message into headers and body but it does not
parse the actual headers or the body.
가능한 HTTP 응용프로그램을 완벽하게 지원하기 위해 Node의 HTTP API는 매우 낮은 수준
입니다. 그것은 스트림 핸들링과 메세지 분석을 다루고 있다. 분석은 메세지 Header와 Body
로 나뉘지만 실제 Header와 Body는 분석하지 않는다.

HTTPS is supported if OpenSSL is available on the underlying platform.
플랫폼에서 OpenSSL을 사용할 수 있는 경우 HTTPS를 지원한다.

## http.Server

This is an `EventEmitter` with the following events:
이것은 다음 이벤트를 갖는 `EventEmitter`이다.

### Event: 'request'

`function (request, response) { }`

 `request` is an instance of `http.ServerRequest` and `response` is
 an instance of `http.ServerResponse`
 `request`는 `http.ServerRequest`의 인스턴스이고 `response`는
 `http.ServerResponse`의 인스턴스이다.

### Event: 'connection'

`function (stream) { }`

 When a new TCP stream is established. `stream` is an object of type
 `net.Stream`. Usually users will not want to access this event. The
 `stream` can also be accessed at `request.connection`.
 새로운 TCP 스트림이 설정되었을 때 `stream`은 `net.Stream` 형식의 개체이다.
 일반적으로 사용자는 이 이벤트에 접근하기를 바라지 않을 것이다. `stream`은
 또한 `request.connection`에 액세스 할 수 있다.

### Event: 'close'

`function (errno) { }`

 Emitted when the server closes.
 서버가 닫힐 때 생성된다.

### Event: 'request'

`function (request, response) {}`

Emitted each time there is request. Note that there may be multiple requests
per connection (in the case of keep-alive connections).
요청마다 생성된다. 하나의 연결당 여러개의 요청이 있을 수 있다는 것을 주의 하세요.
(keep-alive 인 경우)

### Event: 'upgrade'

`function (request, socket, head)`

Emitted each time a client requests a http upgrade. If this event isn't
listened for, then clients requesting an upgrade will have their connections
closed.
클라이언트가 HTTP 업그레이드를 요청할 때마다 생성된다. 이 이벤트가 모니터링 되지 않는 경우
업그레이드를 요청하는 클라이언트의 연결이 종료된다.

* `request` is the arguments for the http request, as it is in the request event.
* `socket` is the network socket between the server and client.
* `head` is an instance of Buffer, the first packet of the upgraded stream, this may be empty.
* `request`는 요청 이벤트와 마찬가지로 HTTP 요청에 대한 인수이다.
* `socket` 는 서버와 클라이언트 사이의 네트워크 소켓이다.
* `head` 는 업데이트된 스트림의 첫번째 패킷을 가진 Buffer의 인스턴스이다. 비어있는 경우도 있다.

After this event is emitted, the request's socket will not have a `data`
event listener, meaning you will need to bind to it in order to handle data
sent to the server on that socket.
이 이벤트가 생성된 후 요청되는 소켓은 이미 `data` 이벤트 리스너가 없다. 이 소켓으로
서버에 보내진 데이터를 사용하기 위해서는 바인딩을 해야한다는 것을 의미한다.

### Event: 'clientError'

`function (exception) {}`

If a client connection emits an 'error' event - it will forwarded here.
클라이언트 연결이 'error' 이벤트를 생성하면 여기에 전송된다.

### http.createServer(requestListener)

Returns a new web server object.
새로운 웹 서버 객체를 반환한다.

The `requestListener` is a function which is automatically
added to the `'request'` event.
`requestListener` 는 `'request'` 이벤트에 자동으로 추가되 함수이다.

### server.listen(port, [hostname], [callback])

Begin accepting connections on the specified port and hostname.  If the
hostname is omitted, the server will accept connections directed to any
IPv4 address (`INADDR_ANY`).
지정된 포트와 호스트의 이름으로 연결을 수락을 시작한다. 호스트 이름이 생략되면
서버는 모든 IPv4 주소에 대한 연결을 허용한다 (`INADDR_ANY`).

To listen to a unix socket, supply a filename instead of port and hostname.
UNIX 소켓을 기다리는 경우, 포트와 호스트 이름이 아닌 파일 이름을 제공한다.

This function is asynchronous. The last parameter `callback` will be called
when the server has been bound to the port.
이 함수는 비동기적이다. 마지막 인수인 `callback` 는 서버 포트에 연결하면 호출된다.


### server.listen(path, [callback])

Start a UNIX socket server listening for connections on the given `path`.
`path`로 주어진 연결을 기다리는 UNIX 소켓 서버를 시작한다.

This function is asynchronous. The last parameter `callback` will be called
when the server has been bound.
이 함수는 비동기적이다. 마지막 인수인 `callback`은 서버가 바인딩될때 호출된다.



### server.setSecure(credentials)

Enables HTTPS support for the server, with the crypto module credentials specifying the private key and certificate of the server, and optionally the CA certificates for use in client authentication.
개인키와 서버 인증서를 지정한 암호 모듈의 인증 정보를 서버에 HTTPS 지원을 활성화한다. 선택적으로 인증기관에서 인증하는 클라이언트 인증을 사용할 수 있다.

If the credentials hold one or more CA certificates, then the server will request for the client to submit a client certificate as part of the HTTPS connection handshake. The validity and content of this can be accessed via verifyPeer() and getPeerCertificate() from the server's request.connection.
인증 정보가 하나 이상의 인증 기관에 있는 경우 서버는 HTTP 연결의 핸드 쉐이크의 일부로 클라이언트 일부로 클라이언트 인증서를 보내도록 클라이언트에 요청한다. 그 효과와 내용은 서버 `request.connect에서 verifyPeer()와 getPeerCertificate()를 통해 접근할 수 있다.

### server.close()

Stops the server from accepting new connections.
서버가 새로운 연결을 받아들이는 것을 멈춘다.


## http.ServerRequest

This object is created internally by a HTTP server--not by
the user--and passed as the first argument to a `'request'` listener.
이 개체는 HTTP 서버 내부 - 사용자가 아닌 - 로 만든 `'request'` 청취자의 
첫번째 인자로 전달된다.

This is an `EventEmitter` with the following events:
이것은 다음 이벤트를 갖는 `EventEmitter`이다.

### Event: 'data'

`function (chunk) { }`

Emitted when a piece of the message body is received.
메시지 본문의 일부를 받은 경우에 생성된다.

Example: A chunk of the body is given as the single
argument. The transfer-encoding has been decoded.  The
body chunk is a string.  The body encoding is set with
`request.setBodyEncoding()`.
예 : 하나의 인자로 본문의 청크가 주어진다. 전송 인코딩은 디코딩
된다. 본문의 청크는 문자열이다. 본문의 인코딩 로
`request.setBodyEncoding()`으로 설정된다.

### Event: 'end'

`function () { }`

Emitted exactly once for each message. No arguments.  After
emitted no other events will be emitted on the request.
메시지에 대해 정확하게 한번 생성된다. 인자는 없다. 이 이벤트가 생성
되면 이 요청에서 생성되는 이벤트는 없다.


### request.method

The request method as a string. Read only. Example:
`'GET'`, `'DELETE'`.
요청 메서드를 나타내는 문자열이다. 참조에만 가능하다. 예:
`'GET'`, '`DELETE'`


### request.url

Request URL string. This contains only the URL that is
present in the actual HTTP request. If the request is:
요청 URL을 나타내는 문자열이다. 이것은 실제로 HTTP 요청에 있는
URL만 포함한다.

    GET /status?name=ryan HTTP/1.1\r\n
    Accept: text/plain\r\n
    \r\n

Then `request.url` will be:
`request.url` 과 같게 된다.

    '/status?name=ryan'

If you would like to parse the URL into its parts, you can use
`require('url').parse(request.url)`.  Example:
URL 요소를 분석하려면, `require('url').parse(request.url)`을 참고하
십시오. 예제

    node> require('url').parse('/status?name=ryan')
    { href: '/status?name=ryan'
    , search: '?name=ryan'
    , query: 'name=ryan'
    , pathname: '/status'
    }

If you would like to extract the params from the query string,
you can use the `require('querystring').parse` function, or pass
`true` as the second argument to `require('url').parse`.  Example:
쿼리 문자열에서 매개 변수를 제거하려면, `require('querystring').parse`
함수를 참조하거나, `require('querystring').parse`의 두번째 인수에 true
를 전달하면 된다. 예제

    node> require('url').parse('/status?name=ryan', true)
    { href: '/status?name=ryan'
    , search: '?name=ryan'
    , query: { name: 'ryan' }
    , pathname: '/status'
    }



### request.headers

Read only.
읽기 전용

### request.httpVersion

The HTTP protocol version as a string. Read only. Examples:
`'1.1'`, `'1.0'`.
Also `request.httpVersionMajor` is the first integer and
`request.httpVersionMinor` is the second.
HTTP 프로토콜 버젼을 나타내는 문자열입니다. 읽기 전용. 예시
`'1.1'`, `'1.0'`.
마찬가지로 `request.httpVersionMajor` 첫번째 정수,
`require.httpVersionMinor` 는 두번째 정수이다.


### request.setEncoding(encoding=null)

Set the encoding for the request body. Either `'utf8'` or `'binary'`. Defaults
to `null`, which means that the `'data'` event will emit a `Buffer` object..
요청 본문의 인코딩을 설정한다.  `'utf8'` 혹은 `'binary'` 중 하나이다. 기본값은 `null`
로 `'data'` 이벤트가 `Buffer`를 생성하는 것을 의미한다.


### request.pause()

Pauses request from emitting events.  Useful to throttle back an upload.
요청에 의한 이벤트 생성을 중단한다. 속도를 떨어뜨리 데 유용하다.

### request.resume()

Resumes a paused request.
중단된 요청을 다시 시작한다.

### request.connection

The `net.Stream` object associated with the connection.
연결과 관련된 `net.Stream` 개체이다.


With HTTPS support, use request.connection.verifyPeer() and
request.connection.getPeerCertificate() to obtain the client's
authentication details.
HTTPS는 request.connection.verifyPeer() 와
request.connection.getPeerCertificate() 클라이언트의 인증 세부
정보를 얻을 수 있다.



## http.ServerResponse

This object is created internally by a HTTP server--not by the user. It is
passed as the second parameter to the `'request'` event. It is a `Writable Stream`.
이 개체는 사용자가 아닌 HTTP 서버 내부적으로 생성된다. `'request'` 리스너 두번째 인자로 전달
된다. 이것은 `Writeable Stream`


### response.writeHead(statusCode, [reasonPhrase], [headers])

Sends a response header to the request. The status code is a 3-digit HTTP
status code, like `404`. The last argument, `headers`, are the response headers.
Optionally one can give a human-readable `reasonPhrase` as the second
argument.
응답 헤더를 보낸다. 상태 코드는 404와 같은 3자리 숫자로 HTTP 상태 코드이다. 마지막 인수
`headers`는 응답 헤더이다. 옵션으로 사람이 읽기 형식 `reasonPhrase` 를 두번째 인자로
전달할 수 있다.

Example:
예제

    var body = 'hello world';
    response.writeHead(200, {
      'Content-Length': body.length,
      'Content-Type': 'text/plain'
    });

This method must only be called once on a message and it must
be called before `response.end()` is called.
이 메소드는 만메세지에 대해서 단 한번 `response.end()` 가 호출되기
전에 호출되어야 한다.

### response.write(chunk, encoding='utf8')

This method must be called after `writeHead` was
called. It sends a chunk of the response body. This method may
be called multiple times to provide successive parts of the body.
이 메소드는 `writeHead` 후에 호출되어야 한다. 이것은 응답 본문의 체크를
보낸다. 이 메소드는 본문의 연속적인 부분을 제공하기 위해 여러번 호출 될 수
도 있다.

`chunk` can be a string or a buffer. If `chunk` is a string,
the second parameter specifies how to encode it into a byte stream.
By default the `encoding` is `'utf8'`.
`chunk`는 문자열 또는 버퍼가 될 수 있다. `chunk`가 문자열인 경우 바이트
스트림 인코딩 방식을 두번째 인자로 지정할 수 있다. 기본 `encoding` 은
`'utf8'` 이다.

**Note**: This is the raw HTTP body and has nothing to do with
higher-level multi-part body encodings that may be used.
주의 : 이것은 원시 HTTP 본문에서 높은 수준의 

The first time `response.write()` is called, it will send the buffered
header information and the first body to the client. The second time
`response.write()` is called, Node assumes you're going to be streaming
data, and sends that separately. That is, the response is buffered up to the
first chunk of body.
처음 `response.write()`를 호출하면 버퍼링되고 있던 헤더와 첫 번째 본문이 클라이언트
로 전송된다. 두번째 `response.write()`가 호출되면, Node는 스트리밍을 분할하여
보내려고 하고 있다고 가정한다. 즉, 응답은 본문의 첫번째 청크로 버퍼링된다.


### response.end([data], [encoding])

This method signals to the server that all of the response headers and body
has been sent; that server should consider this message complete.
The method, `response.end()`, MUST be called on each
response.
이 메소드는 응답의 모든 헤더와 본문을 보낼 것을 서버에 알린다; 서버는 메세지가 종료
했다고 생각한다. 이 `response.end()` 메소드는 각 응답에 대해 호출해야 한다.

If `data` is specified, it is equivalent to calling `response.write(data, encoding)`
followed by `response.end()`.
`data`가 지정된 경우 `response.write(data, encoding)`에 이어 `response.end()`를
호출하는 것과 같다.


## http.Client

An HTTP client is constructed with a server address as its
argument, the returned handle is then used to issue one or more
requests.  Depending on the server connected to, the client might
pipeline the requests or reestablish the stream after each
stream. _Currently the implementation does not pipeline requests._
HTTP 클라이언트는 인수로 전달되는 서버 주소에 의해 만들어진 반환된 핸들은
하나 이상의 요청을 실행하는데 사용된다. 연결된 서버에 따라 클라이언트는
파이프라인 요청 또는 해당 스트림후에 스트림을 다시 설정할 수도 있다. 현재의
구현은 파이프라인 요청을 하지 않는다.

Example of connecting to `google.com`:
`google.com`에 접속하는 예제

    var http = require('http');
    var google = http.createClient(80, 'www.google.com');
    var request = google.request('GET', '/',
      {'host': 'www.google.com'});
    request.end();
    request.on('response', function (response) {
      console.log('STATUS: ' + response.statusCode);
      console.log('HEADERS: ' + JSON.stringify(response.headers));
      response.setEncoding('utf8');
      response.on('data', function (chunk) {
        console.log('BODY: ' + chunk);
      });
    });

There are a few special headers that should be noted.
몇개의 특별한 헤더가 있다는 것을 주의해야 한다.

* The 'Host' header is not added by Node, and is usually required by
  website.
* 'Host' 헤더는 Node가 추가하지만 보통 웹 사이트에서 필요하다.

* Sending a 'Connection: keep-alive' will notify Node that the connection to
  the server should be persisted until the next request.
* 'Connection: keep-alive' 를 보내면 서버에 연결을 다음 요청까지 유지하도록 Node에
알린다.

* Sending a 'Content-length' header will disable the default chunked encoding.
* 'Content-length' 헤더를 전송하기 위한 기본 청크가 비활성화 된다.


### Event: 'upgrade'

`function (request, socket, head)`

Emitted each time a server responds to a request with an upgrade. If this event
isn't being listened for, clients receiving an upgrade header will have their
connections closed.
서버가 업그레이드 요청에 응답 할때마다 생성된다. 이 이벤트를 모니터링하지 않으면 클라이언트가
업그레이드 헤더를 수신하면 해당 연결이 닫힌다.

See the description of the `upgrade` event for `http.Server` for further details.
더 자세한 사항은 `http.Serer`의 `upgrade` 이벤트 설명을 참조하세요.

### http.createClient(port, host='localhost', secure=false, [credentials])

Constructs a new HTTP client. `port` and
`host` refer to the server to be connected to. A
stream is not established until a request is issued.
새로운 HTTP 클라이언트를 생성한다. `port`와 `host`는 연결할
서버를 참조한다. 요청이 되기전에 스트림은 생성되지 않는다.

`secure` is an optional boolean flag to enable https support and `credentials` is an optional credentials object from the crypto module, which may hold the client's private key, certificate, and a list of trusted CA certificates.
`secure` 옵션는 boolean 플래그로 HTTPS 지원을 활성화하고 `credentials`은 암호 모듈의 인증 정보 개체에서 클라이언트 개인키, 인증서 및 신뢰할 수 있는 인증 기관의 인증서 목록을 포함할 수 있다.

If the connection is secure, but no explicit CA certificates are passed in the credentials, then node.js will default to the publicly trusted list of CA certificates, as given in http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt
연결이 안전한 경우 인증 정보를 인증 기관의 인증서를 명시적으로 전달되지 않으면 node.js는 기본적으로 신뢰할 수 있는 인증 기관 목록으로 http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt 을 준다.

### client.request(method='GET', path, [request_headers])

Issues a request; if necessary establishes stream. Returns a `http.ClientRequest` instance.
요청을 실행한다. 필요한 경우 스트림을 설정한다. `http.ClientRequest`는 인스턴스를 반환한다.

`method` is optional and defaults to 'GET' if omitted.
`method`는 옵션에서 생략된 경우 기본은 `GET` 이다.

`request_headers` is optional.
Additional request headers might be added internally
by Node. Returns a `ClientRequest` object.
`request_headers` 는 옵션이다. Node 내부에 추가적인 요청 헤더를
추가하는 경우가 있다. `ClientRequest` 객체를 반환한다.

Do remember to include the `Content-Length` header if you
plan on sending a body. If you plan on streaming the body, perhaps
set `Transfer-Encoding: chunked`.
본문을 전송하려는 경우 `Content-Length` 헤더를 포함하는 것을 잊지 마세요.
본문을 스트림으로 생성하는 경우는 아마도 `Transfer-Encoding: chunked`를
설정하세요.

*NOTE*: the request is not complete. This method only sends the header of
the request. One needs to call `request.end()` to finalize the request and
retrieve the response.  (This sounds convoluted but it provides a chance for
the user to stream a body to the server with `request.write()`.)
*참고*: 요청이 완료되지 않는다.  이 메소드는 요청의 헤더를 보낼 뿐이다. 요청을 완료하여
응답을 읽으려면 request.end()를 호출 해야한다.(복잡하게 느낄지도 모르지만, 이것은
`request.write()` 에 본문을 스트림하는 기회를 사용자에게 제공한다.

### client.verifyPeer()

Returns true or false depending on the validity of the server's certificate in the context of the defined or default list of trusted CA certificates.
지정된 또는 기본적으로 신뢰할 수 있는 인증 기관의 인증서에서 서버 인증서의 유효성에 따라 true 또는 false를 반환한다.

### client.getPeerCertificate()

Returns a JSON structure detailing the server's certificate, containing a dictionary with keys for the certificate 'subject', 'issuer', 'valid\_from' and 'valid\_to'
서버 인증서의 세부 사항을 'subject', 'issue', 'valid_from' 그리고 'vaild_to'를 키와 인증서의 사전을 포함하여 JSON 형식으로 반환한다.


## http.ClientRequest

This object is created internally and returned from the `request()` method
of a `http.Client`. It represents an _in-progress_ request whose header has
already been sent.
이 개체는 HTTP 서버 내부에서 만들어 `http.Client`의 `request()` 메소드에서 반환된다.
그것은 헤더를 보낸 진행중인 요청을 표현한다.

To get the response, add a listener for `'response'` to the request object.
`'response'` will be emitted from the request object when the response
headers have been received.  The `'response'` event is executed with one
argument which is an instance of `http.ClientResponse`.
응답을 검색하려면 `'response'`에 대한 리스너를 요청 개체에 추가한다.`'response'`
이벤트는 응답 헤더를 수신하는 요청 개체가 생성된다. `'response'` 이벤트는
`http.ClientResponse` 인스턴스를 유일한 인수로 실행된다.

During the `'response'` event, one can add listeners to the
response object; particularly to listen for the `'data'` event. Note that
the `'response'` event is called before any part of the response body is received,
so there is no need to worry about racing to catch the first part of the
body. As long as a listener for `'data'` is added during the `'response'`
event, the entire body will be caught.
`'response'` 이벤트 동안 응답 객체에 리스너를 추가할 수 있다. 특히 `'data'` 이벤트를
청취하기 위해서이다. `'response'` 이벤트는 응답 본문의 첫 부분의 수신과 충돌하는 것을
걱정할 필요가 없다. `'response'` 이벤트 사이에 `'data'` 이벤트 리스너가 추가되는 경
우 본문 전체를 받을 수 있다.

    // Good
    request.on('response', function (response) {
      response.on('data', function (chunk) {
        console.log('BODY: ' + chunk);
      });
    });

    // Bad - misses all or part of the body
    request.on('response', function (response) {
      setTimeout(function () {
        response.on('data', function (chunk) {
          console.log('BODY: ' + chunk);
        });
      }, 10);
    });

This is a `Writable Stream`.
이것은 `Writable Stream` 이다.

This is an `EventEmitter` with the following events:
이것은 다음 이벤트를 갖는 `EventEmitter` 이다.

### Event 'response'

`function (response) { }`

Emitted when a response is received to this request. This event is emitted only once. The
`response` argument will be an instance of `http.ClientResponse`.
이 요청에 대한 응답을 수신했을 때 발생한다. 이 이벤트는 단 한번만 생성된다.
`response` 인수는 `http.ClientResponse`의 인스턴스이다.


### request.write(chunk, encoding='utf8')

Sends a chunk of the body.  By calling this method
many times, the user can stream a request body to a
server--in that case it is suggested to use the
`['Transfer-Encoding', 'chunked']` header line when
creating the request.
본문의 청크를 보낸다. 이 메서드를 여러번 호출하여 서버에서
요청 본문을 스트림화 할 수 있다. 이 경우
`['Transfer-Encoding', 'chunked']` 헤더 요청을 생성하
는 것을 의미한다.

The `chunk` argument should be an array of integers
or a string.
`chunk` 인수는 정수 배열 또는 문자열이다.

The `encoding` argument is optional and only
applies when `chunk` is a string.
`encoding` 인수는 선택 사항이며 `chunk`가 문자열의
경우에만 적용된다.


### request.end([data], [encoding])

Finishes sending the request. If any parts of the body are
unsent, it will flush them to the stream. If the request is
chunked, this will send the terminating `'0\r\n\r\n'`.
요청 전송을 종료합니다. 본문의 어떤 부분이 아직 전송되지 않은 경우,
그것은 스트림에 플러시 된다. 요청이 청크되는 경우, 이것은 종단의
`'0\r\n\r\n'`을 보낸다.

If `data` is specified, it is equivalent to calling `request.write(data, encoding)`
followed by `request.end()`.
`data`가 지정된 경우 `request.write(data, encoding)` 에 이어 `reuqest.end()`를 호출하는
것과 같다.


## http.ClientResponse

This object is created when making a request with `http.Client`. It is
passed to the `'response'` event of the request object.
이 개체는 `http.Client` 와 요청을 통해 생성된다.  이것은 요청 객체의 `response`
이벤트에 전달된다.

The response implements the `Readable Stream` interface.
응답은 `Readable Stream`을 구현한다.

### Event: 'data'

`function (chunk) {}`

Emitted when a piece of the message body is received.
메시지 본문의 일부를 받은 경우에 생성된다.

    Example: A chunk of the body is given as the single
    argument. The transfer-encoding has been decoded.  The
    body chunk a String.  The body encoding is set with
    `response.setBodyEncoding()`.
    예: 본문의 청크는 하나의 인수로 주어진다. 전송 인코딩은 디코딩
    된다. 본문의 청크는 문자열이다. 본문 인코딩은 `response.setBodyEncoding()` 으로 설정된다.

### Event: 'end'

`function () {}`

Emitted exactly once for each message. No arguments. After
emitted no other events will be emitted on the response.
메세지에 대해 정확히 한번만 생성된다. 이 이벤트가 생성되면 이 응답
은 어떤 이벤트도 발생하지 않는다.

### response.statusCode

The 3-digit HTTP response status code. E.G. `404`.
3자리의 숫자로 응답 상태 코드이다. 예를 들면 `404`

### response.httpVersion

The HTTP version of the connected-to server. Probably either
`'1.1'` or `'1.0'`.
Also `response.httpVersionMajor` is the first integer and
`response.httpVersionMinor` is the second.
연결된 서버와 HTTP 버전이다. 아마 `'1.1'` 또는 `'1.0'` 중 하나이다.
마찬가지로 `response.httpVersionMajor` 첫번째 정수
`response.httpVersionMinor` 는 2번째 정수이다.

### response.headers

The response headers object.
응답 헤더 객체이다.

### response.setEncoding(encoding=null)

Set the encoding for the response body. Either `'utf8'`, `'ascii'`, or `'base64'`.
Defaults to `null`, which means that the `'data'` event will emit a `Buffer` object..
응답 본문의 인코딩을 설정한다. `'utf8'`, `'ascii'`, `'base64'` 중의 하나이다. 기본값은 `null`
로 `'data'` 이벤트가 `'Buffer` 를 생성하는 것을 의미한다.

### response.pause()

Pauses response from emitting events.  Useful to throttle back a download.
이벤트를 생성하여 응답을 중단한다. 다운로드 속도를 떨어트리는데 유용하다.

### response.resume()

Resumes a paused response.
중단된 응답을 다시 시작한다.

### response.client

A reference to the `http.Client` that this response belongs to.
이 응답을 소유한 `http.Client`에 대한 참조이다



## net.Server

This class is used to create a TCP or UNIX server.
이 클래스는 TCP 또는 UNIX 도메인의 서버를 생성하는데 사용된다.

Here is an example of a echo server which listens for connections
on port 8124:
8124 포트를 기다리는 에코 서버 예제

    var net = require('net');
    var server = net.createServer(function (stream) {
      stream.setEncoding('utf8');
      stream.on('connect', function () {
        stream.write('hello\r\n');
      });
      stream.on('data', function (data) {
        stream.write(data);
      });
      stream.on('end', function () {
        stream.write('goodbye\r\n');
        stream.end();
      });
    });
    server.listen(8124, 'localhost');

To listen on the socket `'/tmp/echo.sock'`, the last line would just be
changed to
`'/tmp/echo.sock'`에 소켓을 기다리는 마지막 줄을 이렇게 변경한다.

    server.listen('/tmp/echo.sock');

This is an `EventEmitter` with the following events:
이것은 다음과 같은 이벤트가 있는 `EventEmitter` 이다.

### Event: 'connection'

`function (stream) {}`

Emitted when a new connection is made. `stream` is an instance of
`net.Stream`.
새 연결이 만들어지면 발생한다. `stream` 은 `net.Stream` 의 인스턴스이다.


### Event: 'close'

`function () {}`

Emitted when the server closes.
서버가 닫힐 때 생성된다.


### net.createServer(connectionListener)

Creates a new TCP server. The `connectionListener` argument is
automatically set as a listener for the `'connection'` event.
새로운 TCP 서버를 생성한다.  `connectionListener` 인수는 `'connect'`
이벤트에 대한 리스너로 자동으로 추가된다.


### server.listen(port, [host], [callback])

Begin accepting connections on the specified `port` and `host`.  If the
`host` is omitted, the server will accept connections directed to any
IPv4 address (`INADDR_ANY`).
지정된 `port` 를 `host` 에 연결 접수를 시작한다. `host`가 생략되면 서버는 모든
IPv4 주소에 대한 연결을 허용한다. (`INADDR_ANY`)

This function is asynchronous. The last parameter `callback` will be called
when the server has been bound.
이 함수는 비동기 함수이다. 마지막 인수 `callback`은 서버가 바인딩 되었을 때 호출된다.


### server.listen(path, [callback])

Start a UNIX socket server listening for connections on the given `path`.
주어진 `path`에 연결 수신 대기하는 UNIX 도메인 소켓 서버를 시작한다.
주어진 `path`로 연결 수신 대기중인 UNIX 도메인 소켓 서버를 시작한다.

This function is asynchronous. The last parameter `callback` will be called
when the server has been bound.
이 함수는 비동기 함수이다. 마지막 인수 `callback` 은 서버가 바인딩 되었을 때 호출된다.


### server.listenFD(fd)

Start a server listening for connections on the given file descriptor.
주어진 파일 기술자의 연결을 기다리는 서버를 시작한다.

This file descriptor must have already had the `bind(2)` and `listen(2)` system
calls invoked on it.
이 파일 기술자는 이미 `bind(2)` 및 `listen(2)` 시스템 호출을 호출하지 않으면 안된다.

### server.close()

Stops the server from accepting new connections. This function is
asynchronous, the server is finally closed when the server emits a `'close'`
event.
서버가 새로운 연결을 접근을 멈춘다.  이 함수는 비동기으로 서버가 마지막으로 `'close'`
이벤트를 생성했을 때 닫힌다.

### server.maxConnections

Set this property to reject connections when the server's connection count gets high.
서버 연결이 많아졌을 때 거부하기 위해 이 속성을 설정한다.

### server.connections

The number of concurrent connections on the server.
이 서버의 동시 연결 수 이다.



## net.Stream

This object is an abstraction of of a TCP or UNIX socket.  `net.Stream`
instance implement a duplex stream interface.  They can be created by the
user and used as a client (with `connect()`) or they can be created by Node
and passed to the user through the `'connection'` event of a server.
이 개체는 TCP 또는 UNIX 소켓을 추상화한 것이다. `net.Stream` 인스턴스는 양방향
스트림 인터페이스를 구현한다.  그들은 사용자에 의해 만들어진 (`connect()`에 의해)
클라이언트로 사용되거나, Node에 의해 만들어진 서버 `connection` 이벤트를 통해 사용자
에게 전달된다.


`net.Stream` instances are EventEmitters with the following events:
`net.Stream` 인스턴스는 다음과 같은 이벤트를 갖는 EventEmitter이다.

### Event: 'connect'

`function () { }`

Emitted when a stream connection successfully is established.
See `connect()`.
?스트림 연결이 성공되었을 경우에 생성된다. `connect()`를 참고하세요.


### Event: 'secure'

`function () { }`

Emitted when a stream connection successfully establishes an SSL handshake with its peer.
?스트림 연결 대상과 SSL 핸드 쉐이크의 설치가 성공했을 경우 생성된다.


### Event: 'data'

`function (data) { }`

Emitted when data is received.  The argument `data` will be a `Buffer` or
`String`.  Encoding of data is set by `stream.setEncoding()`.
(See the section on `Readable Stream` for more information.)
데이터를 받을때 생성된다.  `data` 인자는 `Buffer` 또는 `String`이다. 데이터 인코
딩은 `stream.setEncoding()`으로 설정된다. (더 자세한 정보는 `Readable Stream`을
참고하세요.)

### Event: 'end'

`function () { }`

Emitted when the other end of the stream sends a FIN packet. After this is
emitted the `readyState` will be `'writeOnly'`. One should probably just
call `stream.end()` when this event is emitted.
스트림의 상대가 FIN 패킷을 보낼때 생성된다. 이 이벤트가 생성된 후 `readyState`는
`'writeOnly'`이다. 이 이벤트가 생성되면 아마 `stream.end()`를 호출해야 할 것이다.

### Event: 'timeout'

`function () { }`

Emitted if the stream times out from inactivity. This is only to notify that
the stream has been idle. The user must manually close the connection.
스트림이 시간 초과하여 비활성될 때 생성된다. 이것은 스트림이 유휴 상태가 된것을 알리는
것이다. 이용자는 수동으로 연결을 종료해야 한다.

See also: `stream.setTimeout()`
관련 항목 : `stream.setTimeout()`


### Event: 'drain'

`function () { }`

Emitted when the write buffer becomes empty. Can be used to throttle uploads.
쓰기 버퍼가 비어있는 경우에 생성된다. 업로드 속도를 떨어뜨리는데 사용할 수 있다.

### Event: 'error'

`function (exception) { }`

Emitted when an error occurs.  The `'close'` event will be called directly
following this event.
오류가 발생한 경우에 생성된다. `'close'` 이벤트는 이 이벤트 다음에 직접 호출된다.

### Event: 'close'

`function (had_error) { }`

Emitted once the stream is fully closed. The argument `had_error` is a boolean which says if
the stream was closed due to a transmission
error.
스트림이 완전히 닫힌 경우에 생성된다. 인수 `had_error`는 boolean 으로 스트림 전송 오류에 의해서 종료
되었는지에 대한 여부를 나타낸다.


### net.createConnection(port, host='127.0.0.1')

Construct a new stream object and opens a stream to the specified `port`
and `host`. If the second parameter is omitted, localhost is assumed.
새 스트림 개체를 생성하고 지정된 `port`로 `host`에 스트림을 연다. 두번째 인수를
생략하면 로컬 호스트로 가정한다.

When the stream is established the `'connect'` event will be emitted.
스트림이 설정되면 `'connect'` 이벤트가 생성된다.

### stream.connect(port, host='127.0.0.1')

Opens a stream to the specified `port` and `host`. `createConnection()`
also opens a stream; normally this method is not needed. Use this only if
a stream is closed and you want to reuse the object to connect to another
server.
지정된 `port`로 `host`에서 스트림을 연다. `createConnection()` 또한 스트림을
연다. : 일반적으로는 이 메소드를 필요로 하지 않는다. 이것을 사용하는 경우는 스트
림이 닫힌 후 다른 서버로 다시 연결할 경우에 사용한다.

This function is asynchronous. When the `'connect'` event is emitted the
stream is established. If there is a problem connecting, the `'connect'`
event will not be emitted, the `'error'` event will be emitted with 
the exception.
이 함수는 비동기적이다. 스트림이 설정되면 `'connect'` 이벤트가 생성된다. 연결에
문제가 있을 경우 `connect`이벤트가 생성되지 않고, 예외와 함께 `'error'` 이벤트가
생성된다.


### stream.remoteAddress

The string representation of the remote IP address. For example,
`'74.125.127.100'` or `'2001:4860:a005::68'`.
원격 IP주소를 표현하는 문자열이다. 예를 들어 `'74:125:127:100'` 혹은
`'2001:4860:a005::68'`

This member is only present in server-side connections.
이 맴버는 서버측 연결에 대해서만 부여한다.

### stream.readyState

Either `'closed'`, `'open'`, `'opening'`, `'readOnly'`, or `'writeOnly'`.
`'closed'`, `'open'`, `'opening'`, `'readOnly'` 또는 `'writeOnly'` 중 하나이다.

### stream.setEncoding(encoding=null)

Sets the encoding (either `'ascii'`, `'utf8'`, or `'base64'`) for data that is
received.
수신한 데이터의 인코딩을 설정합니다. (`'ascii'`, '`utf8'`, `'base64'` 중에 하나이다.)

### stream.setSecure([credentials])

Enables SSL support for the stream, with the crypto module credentials specifying the private key and certificate of the stream, and optionally the CA certificates for use in peer authentication.
스트림의 개인키와 서버 인증서로 지정한 암호 모듈의 인증정보를 스트림 인증 정보에 대해 SSL 지원을 활성화한다. 선택적으로 인증 기관에서 입증된 상대방의 인증을 사용할 수 있다.

If the credentials hold one ore more CA certificates, then the stream will request for the peer to submit a client certificate as part of the SSL connection handshake. The validity and content of this can be accessed via verifyPeer() and getPeerCertificate().
인증 정보가 하나 이상의 인증 기관의 인증서가 있는 경우, 스트림은 SSL 연결 핸드 쉐이크(SSL handshake)의 일부로 클라이언트 인증서를 보내도록 상대에게 요구한다. 그 유효성과 내용은 verifyPeer()와 getPeerCertificate()를 통해 엑세스할 수 있다.

### stream.verifyPeer()

Returns true or false depending on the validity of the peers's certificate in the context of the defined or default list of trusted CA certificates.
지정된 또는 기본 신뢰할 수 있는 인증 기관의 인증서에서 상대방이 인증서의 유효성에 따라 true 또는 false를 반환한다.

### stream.getPeerCertificate()

Returns a JSON structure detailing the peer's certificate, containing a dictionary with keys for the certificate 'subject', 'issuer', 'valid\_from' and 'valid\_to'
상대의 인증서 정보를, 'subject', 'issuer', 'valid_form' 그리고 'valid_to'를 키와 인증서의 사전을 포함하여 JSON 형식으로 반환한다.

### stream.write(data, encoding='ascii')

Sends data on the stream. The second parameter specifies the encoding in
the case of a string--it defaults to ASCII because encoding to UTF8 is rather
slow.
스트림에 데이터를 전송한다.  문자열의 경우 두번째 인수는 인코딩을 지정한다. UTF8은 좀더
느리므로 기본값은 ASCII이다.

Returns `true` if the entire data was flushed successfully to the kernel
buffer. Returns `false` if all or part of the data was queued in user memory.
`'drain'` will be emitted when the buffer is again free.
데이터 전체가 커널 버퍼에 플러쉬가 성공하면 `true`를 반환한다. 데이터 일부 또는 전체가
사용자 메모리의 대기열에 넣은 경우에는 `false`를 반환한다. 버퍼가 다시 비는 경우
`'drain'` 이벤트가 생성된다.

### stream.end([data], [encoding])

Half-closes the stream. I.E., it sends a FIN packet. It is possible the
server will still send some data. After calling this `readyState` will be
`'readOnly'`.
반쯤 닫힌 스트림이다. 예를 들어 FIN 패킷을 보낸다.  서버가 데이터를 계속 보내고
있을 수 있다. 이 메소드를 호출한 후 `readyState`는 `'readOnly'`가 됩니다.

If `data` is specified, it is equivalent to calling `stream.write(data, encoding)`
followed by `stream.end()`.
`data`가 지정된 경우 `stream.write(data, encoding)`에 이어 `stream.end()`을 호출하는
것과도 같다.

### stream.destroy()

Ensures that no more I/O activity happens on this stream. Only necessary in
case of errors (parse error or so).
이 스트림에서 어떤 I/O도 일어나지 않는다는 것을 보장한다. (해석 오류등) 오류의 경우에만 필요하다.

### stream.pause()

Pauses the reading of data. That is, `'data'` events will not be emitted.
Useful to throttle back an upload.
데이터 로드를 중단한다. 즉, `'data'` 이벤트가 생성되지 않는다. 업로드 속도를 떨어
트리는데 유용하다.

### stream.resume()

Resumes reading after a call to `pause()`.
`pause()`를 호출한 후에 다시 로드를 시작한다.

### stream.setTimeout(timeout)

Sets the stream to timeout after `timeout` milliseconds of inactivity on
the stream. By default `net.Stream` do not have a timeout.
비활성 스트림이 `timeout` 밀리초 후에 동작하도록 설정한다. 기본적으로 `net.Stream`
타이암아웃으로 동작하지 않는다.

When an idle timeout is triggered the stream will receive a `'timeout'`
event but the connection will not be severed. The user must manually `end()`
or `destroy()` the stream.
유휴 상태가 발생하면 스트림 `'timeout'`이 발생하지만 연결은 종료되지 않는다. 사용자는
수동으로 end() 또는 destory()를 호출해야한다.

If `timeout` is 0, then the existing idle timeout is disabled.
`timeout`이 0이면 유휴 상태가 해제된다.

### stream.setNoDelay(noDelay=true)

Disables the Nagle algorithm. By default TCP connections use the Nagle
algorithm, they buffer data before sending it off. Setting `noDelay` will
immediately fire off data each time `stream.write()` is called.
Nagle 알고리즘을 비활성화한다. 기본적으로 TCP 연결은 Nagle 알고리즘을 사용하는데
데이터를 전송하기 전에 버퍼링을 한다. noDelay로 설정하면 데이터가 `stream.write()`를
호출 할 때마다 즉시 전송한다.

### stream.setKeepAlive(enable=false, [initialDelay])

Enable/disable keep-alive functionality, and optionally set the initial
delay before the first keepalive probe is sent on an idle stream.
Set `initialDelay` (in milliseconds) to set the delay between the last
data packet received and the first keepalive probe. Setting 0 for
initialDelay will leave the value unchanged from the default
(or previous) setting.
연결 유지 기능을 활성화/비활성화한다. 선택적으로 연결 유지의 첫 probe가 유휴
스트림에 보낼 때까지 초기 지연을 설정한다.  `initialDelay`(밀리초)로 설정하면
마지막 데이터 패킷을 수신하고 연결 유지 probe 까지 지연이 설정된다. 초기 지연에
0을 설정하면 기본 설정에서 값을 변경되지 않도록 한다.



## Crypto

Use `require('crypto')` to access this module.
이 모듈에 액세스하려면 `require('crypto')` 를 사용하세요.

The crypto module requires OpenSSL to be available on the underlying platform.
It offers a way of encapsulating secure credentials to be used as part of a secure
HTTPS net or http connection.
암호화 모듈은 하부의 플랫폼에서 OpenSSL 을 사용할 것을 요구한다. 그것은 안전한 HTTPS 네트워크와 http 연결의 일부로 사용되는 안전한 인증 정보를 캡슐화하는 방법을 제공한다.

It also offers a set of wrappers for OpenSSL's hash, hmac, cipher, decipher, sign and verify methods.
동시에 OpenSSL 해시, HMAC, 암호화, 복호화 서명 그리고 검증에 래퍼를 일체 제공한다.

### crypto.createCredentials(details)

Creates a credentials object, with the optional details being a dictionary with keys:
인증 정보 개체를 만든다. 옵션 details 다음 키가 있는 사전이다.

`key` : a string holding the PEM encoded private key
`key` : PEM 인코딩된 비밀키를 포함하는 문자열

`cert` : a string holding the PEM encoded certificate
`cert` : PEM 인코딩된 인증서를 포함하는 문자열

`ca` : either a string or list of strings of PEM encoded CA certificates to trust.
`ca` : 신뢰할 수 있는 인증 기관의 인증서가 PEM으로 인코딩된 문자열 또는 문자열 배열

If no 'ca' details are given, then node.js will use the default publicly trusted list of CAs as given in 
http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt
`ca` 정보가 주어지지 않는 경우 node.js 는 기본적으로 http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt
에 주어진 신뢰할 수 있는 인증 기관의 공개 목록을 사용한다.


### crypto.createHash(algorithm)

Creates and returns a hash object, a cryptographic hash with the given algorithm which can be used to generate hash digests.
해시 객체를 생성하여 반환한다. 주어진 알고리즘으로 암호화 해시 함수는 다이제스트를 생성하는데 사용한다.

`algorithm` is dependent on the available algorithms supported by the version of OpenSSL on the platform. Examples are sha1, md5, sha256, sha512, etc. On recent releases, `openssl list-message-digest-algorithms` will display the available digest algorithms.
`alogrithm` 은 플랫폼에서 OpenSSL 버전에서 지원되는 사용 가능한 알고리즘에 의존한다. 예를 들어 sha1, md5, sha256, sha512 등 이다. 최신 릴리즈에서는 `openssl list-message-digest-algorithm`에서 사용할 수 있는 다이제스트 알고리즘이 표시된다.

### hash.update(data)

Updates the hash content with the given `data`. This can be called many times with new data as it is streamed.
?주어진 `data`에서 해시의 내용을 업데이트 한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우

### hash.digest(encoding='binary')

Calculates the digest of all of the passed data to be hashed. The `encoding` can be 'hex', 'binary' or 'base64'.
전달된 모든 데이터가 해시 다이제스트를 계산한다. `encoding`은 'hex', 'binary' 또는 'base64' 중 하나이다.

### crypto.createHmac(algorithm, key)

Creates and returns a hmac object, a cryptographic hmac with the given algorithm and key.
주어진 키와 알고리즘으로 암호화된 hmac 객체를 생성하여 반환한다.
                                                                                                                                                                                                      `
`algorithm` is dependent on the available algorithms supported by OpenSSL - see createHash above.
`key` is the hmac key to be used.
`algorithm` 은 OpenSSL에서 지원하는 알고리즘에 따라 달라진다. - 위에서 언급한 createHash 를 참고하세요.
`key`는 hmac 키를 사용한다.

### hmac.update(data)

Update the hmac content with the given `data`. This can be called many times with new data as it is streamed.
주어진 `data`로 HMAC 내용을 업데이트 한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우 여러번 호출된다.


### hmac.digest(encoding='binary')

Calculates the digest of all of the passed data to the hmac. The `encoding` can be 'hex', 'binary' or 'base64'.
전달된 모든 데이터가 HMAC화된 다이제스트를 계산한다. `encoding`은 'hex', 'birnay' 또는 'base64' 중 하나이다.


### crypto.createCipher(algorithm, key)

Creates and returns a cipher object, with the given algorithm and key.
주어진 알고리즘과 키를 사용하여 암호화 개체를 만들어 반환한다.

`algorithm` is dependent on OpenSSL, examples are aes192, etc. On recent releases, `openssl list-cipher-algorithms` will display the available cipher algorithms.
`algorithm`은 OpenSSL에 의존한다. 예를 들어 aes192 등을 말한다. 최신 릴리즈에 `oepnssl list-cipher-algorithms`에서 사용할 수 있는 암호화 알고리즘이 표시된다.

### cipher.update(data, input_encoding='binary', output_encoding='binary')

Updates the cipher with `data`, the encoding of which is given in `input_encoding` and can be 'utf8', 'ascii' or 'binary'. The `output_encoding` specifies the output format of the enciphered data, and can be 'binary', 'base64'  or 'hex'.
`date` 암호를 업데이트 한다. `input_encodind` 에 주어진 인코딩은 `utf8`, `ascii` 혹은 `binary` 이다. `output_encoding`는 암호화된 데이터의 출력로 형식을 지정하는 것으로, `binary`, `base64`, `hex` 중의 하나이다.


Returns the enciphered contents, and can be called many times with new data as it is streamed.
암호호된 컨텐츠를 반환한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우 여러번 호출된다.

### cipher.final(output_encoding='binary')

Returns any remaining enciphered contents, with `output_encoding` being one of: 'binary', 'ascii' or 'utf8'.
암호화된 콘텐츠의 나머지를 반환한다. `output_encoding`는 다음중 하나일 수 있다. 'binary', 'ascii' 또는 'utf8'

### crypto.createDecipher(algorithm, key)

Creates and returns a decipher object, with the given algorithm and key. This is the mirror of the cipher object above.
주어진 알고리즘과 키를 사용하여 암호 개체를 반환한다. 이것은 위에서 언급한 암호 개체의 사본이다.

### decipher.update(data, input_encoding='binary', output_encoding='binary')

Updates the decipher with `data`, which is encoded in 'binary', 'base64' or 'hex'. The `output_decoding` specifies in what format to return the deciphered plaintext - either 'binary', 'ascii' or 'utf8'.
`binary`, `base64` 또는 'hex'중 하나로 인코딩 된 암호를 `data`로 업데이트 한다. `output_decoding`이 암호화된 텍스트 형식을 지정하는 것으로, 'binary', 'ascii' 혹은 'utf8'중에 하나이다.

### decipher.final(output_encoding='binary')

Returns any remaining plaintext which is deciphered, with `output_encoding' being one of: 'binary', 'ascii' or 'utf8'.
암호화된 텍스트의 나머지를 반환한다. `output_encoding` 은 `binary','ascii' 혹은 'utf8'중의 하나이다.


### crypto.createSign(algorithm)

Creates and returns a signing object, with the given algorithm. On recent OpenSSL releases, `openssl list-public-key-algorithms` will display the available signing algorithms. Examples are 'RSA-SHA256'.
주어진 알고리즘으로 서명 객체를 만들어 반환한다. 최근 OpenSSL 버전에서는 `openssl list-puglic-key-algorithms`에서 사용할 수 있는 서명 알고리즘의 목록이 표시된다. 예를 들어 'RSA-SHA256'

### signer.update(data)

Updates the signer object with data. This can be called many times with new data as it is streamed.
서명 객체를 데이터로 업데이트 한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우 여러번 호출된다.

### signer.sign(private_key, output_format='binary')

Calculates the signature on all the updated data passed through the signer. `private_key` is a string containing the PEM encoded private key for signing.
서명 객체에 전달된 모든 갱신 데이터 서명을 계산한다. `private_key`는 PEM 인코딩된 비밀키를 내용으로 하는 문자열이다.

Returns the signature in `output_format` which can be 'binary', 'hex' or 'base64'
'binary', 'hex' 혹은 'base64'중 하나를 지정한 `output_format` 서명을 반환한다.

### crypto.createVerify(algorithm)

Creates and returns a verification object, with the given algorithm. This is the mirror of the signing object above.
주어진 알고리즘으로 검증 객체를 만들어 반환한다. 이것은 이전의 서명 개체와 똑같은 복사본이다.

### verifier.update(data)

Updates the verifyer object with data. This can be called many times with new data as it is streamed.
유효성 검사 개체를 업데이트 한다. 이것은 새로운 데이터 스트림으로 흘러가는 경우 여러번 호출된다.

### verifier.verify(public_key, signature, signature_format='binary')

Verifies the signed data by using the `public_key` which is a string containing the PEM encoded public key, and `signature`, which is the previously calculates signature for the data, in the `signature_format` which can be 'binary', 'hex' or 'base64'.
서명된 데이터를 `public_key`와 `signature`에서 확인한다. `public_key`는 PEM으로 인코딩된 공개키를 포함하는 문자열입니다. `signature` 는 먼저 계산하여 데이터 서명이 `signature_format`은 `binary`, 'hex', 'base64' 중 하나이다.

Returns true or false depending on the validity of the signature for the data and public key.
서명된 데이터와 공개키의 검증 결과에 따라 true 또는 false를 반환한다.



## DNS

Use `require('dns')` to access this module.
이 모듈에 엑세스하려면 `require('dns')`를 사용한다.

Here is an example which resolves `'www.google.com'` then reverse
resolves the IP addresses which are returned.
여기, `'www.google.com'`을 풀이(resolve)하여, 반환되는 IP주소를 역방향
풀이(reverse resolves)하는 하는 예가 있다.

   var dns = require('dns');

    dns.resolve4('www.google.com', function (err, addresses) {
      if (err) throw err;

      console.log('addresses: ' + JSON.stringify(addresses));

      addresses.forEach(function (a) {
        dns.reverse(a, function (err, domains) {
          if (err) {
            console.log('reverse for ' + a + ' failed: ' +
              err.message);
          } else {
            console.log('reverse for ' + a + ': ' +
              JSON.stringify(domains));
          }
        });
      });
    });

### dns.lookup(domain, family=null, callback)

Resolves a domain (e.g. `'google.com'`) into the first found A (IPv4) or
AAAA (IPv6) record.
도메인 (예 `'google.com'`)를 풀이(resolve)하여 처음 발견된 A (IPv4) 또는
AAAA (IPv6) 기록한다.

The callback has arguments `(err, address, family)`.  The `address` argument
is a string representation of a IP v4 or v6 address. The `family` argument
is either the integer 4 or 6 and denotes the family of `address` (not
neccessarily the value initially passed to `lookup`).
콜백은 인수 `(err, address, family)` 를 가지고 있다. `address` 인수는 IP v4 또는
v6 주소 표기 문자열이다. `family` 인수는 4,6 정수 중 하나이고 `address` 와 쌍이다.
(이 값은 반드시 먼저 `lookup` 에 전달할 필요는 없다. )


### dns.resolve(domain, rrtype='A', callback)

Resolves a domain (e.g. `'google.com'`) into an array of the record types
specified by rrtype. Valid rrtypes are `A` (IPV4 addresses), `AAAA` (IPV6
addresses), `MX` (mail exchange records), `TXT` (text records), `SRV` (SRV
records), and `PTR` (used for reverse IP lookups).
rrtype이 명시된 레코드 타입 배열의 도메인(예 `'google.com'`) 풀이. 유효한 rrtype은
`A` (IPV4 주소) `AAAA` (IPV6 주소) `MX` (mail exchange 레코드), `TXT`
(텍스트 레코드), `SRV` (SRV 레코드), `PTR` (IP를 역방향으로 조회하는 데 사용된다) 이다.

The callback has arguments `(err, addresses)`.  The type of each item
in `addresses` is determined by the record type, and described in the
documentation for the corresponding lookup methods below.
콜백은 인수 `(err, addresses)` 를 가진다. `addresses` 각 항목의 타입은 레코드 타입에
의해 결정되고, 해당조회 방법은 아래 문서에 설명되어있다.

On error, `err` would be an instanceof `Error` object, where `err.errno` is
one of the error codes listed below and `err.message` is a string describing
the error in English.
오류 발생시, `err` 는 `Error` 개체의 인스턴스이며, `err.errno` 은 아래 오류 코드 중
하나이고 `err.message` 는 영문 에러 문자열이다.


### dns.resolve4(domain, callback)

The same as `dns.resolve()`, but only for IPv4 queries (`A` records). 
`addresses` is an array of IPv4 addresses (e.g.  
`['74.125.79.104', '74.125.79.105', '74.125.79.106']`).
`dns.resolve()` 과 같다. 단, IPv4 질의(`A` 레코드들)만 지원한다. `addresses` 는
IPv4 주소 배열이다. (예. [`'74.125.79.104'`, `'74.125.79.105'`, `'74.125.79.106'`] ).

### dns.resolve6(domain, callback)

The same as `dns.resolve4()` except for IPv6 queries (an `AAAA` query).
IPv6 질의(`AAAA` 질의) 인것 제외하고는 `dns.resolve4()` 와 동일하다.


### dns.resolveMx(domain, callback)

The same as `dns.resolve()`, but only for mail exchange queries (`MX` records).
`dns.resolve()` 과 같다. 단, mail exchange 질의(`MX` 레코드들)만 지원한다.

`addresses` is an array of MX records, each with a priority and an exchange
attribute (e.g. `[{'priority': 10, 'exchange': 'mx.example.com'},...]`).
`addresses` 는 MX 레코드 배열로, 각각은 `priority`와 e`xchange` 속성을 갇는다.
(예. `[{'priority': 10, 'exchange': 'mx.example.com'},...]`).

### dns.resolveTxt(domain, callback)

The same as `dns.resolve()`, but only for text queries (`TXT` records).
`addresses` is an array of the text records available for `domain` (e.g.,
`['v=spf1 ip4:0.0.0.0 ~all']`).
`dns.resolve()`와 같다. 단, 텍스트 질의(`TXT` 레코드들)만 지원한다. `addresses`는
`domain`에서 가능한 텍스트 레코드 배열이다. (예,  `['v=spf1 ip4:0.0.0.0 ~all']`).

### dns.resolveSrv(domain, callback)

The same as `dns.resolve()`, but only for service records (`SRV` records).
`addresses` is an array of the SRV records available for `domain`. Properties
of SRV records are priority, weight, port, and name (e.g., 
`[{'priority': 10, {'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`).
`dns.resolve()`와 같다. 단, 서비스 레코드(`SRV` 레코드들)만 지원한다. `addresses` 는
`domain`에서 가능한 SRV 레코드들 배열이다. SRV 레코드의 속성은 weight, port, name이다.
(예, `[{'priority': 10, {'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`).

### dns.reverse(ip, callback)

Reverse resolves an ip address to an array of domain names.
IP주소로 도메인명 배열을 역방향 풀이(resolves)한다.

The callback has arguments `(err, domains)`.
콜백은 `(err, domains)` 인수를 가진다.

If there an an error, `err` will be non-null and an instanceof the Error
object.
에러가 발생하면 `err`은 null이 아닌 Error 객체의 인스턴스이다.

Each DNS query can return an error code.
각 DNS 질의는 에러코드를 반환할 수 있다.

- `dns.TEMPFAIL`: timeout, SERVFAIL or similar.
- `dns.TEMPFAIL`: 타임아웃, SERVFAIL 또는 이와 유사한것.
- `dns.PROTOCOL`: got garbled reply.
- `dns.PROTOCOL`: 알수 없는 응답.
- `dns.NXDOMAIN`: domain does not exists.
- `dns.NXDOMAIN`: 도메인이 존재하지 않음
- `dns.NODATA`: domain exists but no data of reqd type.
- `dns.NODATA`: 도메인이 존재하지만, 데이터 없음
- `dns.NOMEM`: 과정에서 메모리가 부족.
- `dns.BADQUERY`: the query is malformed.
- `dns.BADQUERY`: 질의의 형식 오류.


## dgram

Datagram sockets are available through `require('dgram')`.  Datagrams are most commonly 
handled as IP/UDP messages, but they can also be used over Unix domain sockets.
데이터그램 소캣들은 `require('dgram')`에서 사용할 수 있다. 데이터그램은 대부분의 경우 IP/UDP
메시지 처리되지만 UNIX 도메인 소켓을 사용할 수 있다.

### Event: 'message'

`function (msg, rinfo) { }`

Emitted when a new datagram is available on a socket.  `msg` is a `Buffer` and `rinfo` is
an object with the sender's address information and the number of bytes in the datagram.
소켓에 새 데이터 그램이 도착했을 때 생성된다. msg 는 Buffer이고, rinfo는 발신자 주소정보와
데이터그램의 사이즈(bytes)를 가진 객체이다.

### Event: 'listening'

`function () { }`

Emitted when a socket starts listening for datagrams.  This happens as soon as UDP sockets
are created.  Unix domain sockets do not start listening until calling `bind()` on them.
소켓이 데이터그램 수신을 시작할때 생성된다. 이것은 UDP 소켓이 작성되는 즉시 발생한다. UNIX 도메인
소켓은 bind ()를 호출할 때까지 리스닝을 시작하지 않는다.

### Event: 'close'

`function () { }`

Emitted when a socket is closed with `close()`.  No new `message` events will be emitted
on this socket.
close()이 소켓을 받을 때 생성된다. 이 소켓은 새로운 message 이벤트가 생성되지 않는다.

### dgram.createSocket(type, [callback])

Creates a datagram socket of the specified types.  Valid types are:
`udp4`, `udp6`, and `unix_dgram`.
지정된 형식의 데이터그램을 만든다. 유효한 타입은 udp4, udp6, ,unix_dgram이다.

Takes an optional callback which is added as a listener for `message` events.
message 이벤트 리스너가 더해진 콜백으로 이동한다. 필수는 아니다.

### dgram.send(buf, offset, length, path, [callback])

For Unix domain datagram sockets, the destination address is a pathname in the filesystem.
An optional callback may be supplied that is invoked after the `sendto` call is completed
by the OS.  It is not safe to re-use `buf` until the callback is invoked.  Note that 
unless the socket is bound to a pathname with `bind()` there is no way to receive messages
on this socket.
UNIX 도메인의 데이터그램 소켓 용. 수신지 종단 주소는 파일 시스템의 경로명이다. 옵션인 콜백은 OS에서
sendto 호출이 완료된 후에 시작되기 위하여 제공된다. 콜백이 호출될 때까지 buf 재사용은 안전하지 않다.
bind()로 소켓이 경로명에 바인딩되지 않는 한, 이 소켓으로 메시지를 받을 수 없다는 것에 주의.

Example of sending a message to syslogd on OSX via Unix domain socket `/var/run/syslog`:
Unix 도메인소켓 /var/run/syslog로 OSX에서 syslogd에 메세지를 보내는 예제:

    var dgram = require('dgram');
    var message = new Buffer("A message to log.");
    var client = dgram.createSocket("unix_dgram");
    client.send(message, 0, message.length, "/var/run/syslog",
      function (err, bytes) {
        if (err) {
          throw err;
        }
        console.log("Wrote " + bytes + " bytes to socket.");
    });

### dgram.send(buf, offset, length, port, address, [callback])

For UDP sockets, the destination port and IP address must be specified.  A string
may be supplied for the `address` parameter, and it will be resolved with DNS.  An 
optional callback may be specified to detect any DNS errors and when `buf` may be
re-used.  Note that DNS lookups will delay the time that a send takes place, at
least until the next tick.  The only way to know for sure that a send has taken place
is to use the callback.
UDP 소켓 용. 수신지 종단의 포트와 IP주소는 필수다. address인자에 문자열을 넘기면, DNS와
해결된다. DNS 오류 발생시 와 buf가 재사용 가능하게 된 경우를 위해 콜백을 지정할 수 있다.
콜백은 옵션이다. DNS 조회는 발신시간을 (적어도 다음 Tick 까지) 지연시킬것이란 걸 참고하라.
발신을 했다는 것을 확실히 아는 유일한 방법은 콜백을 사용하는 것이다.

Example of sending a UDP packet to a random port on `localhost`;
localhost 에서 임의의 포트로 UDP 패킷을 발신하는 예제.

    var dgram = require('dgram');
    var message = new Buffer("Some bytes");
    var client = dgram.createSocket("udp4");
    client.send(message, 0, message.length, 41234, "localhost");
    client.close();


### dgram.bind(path)

For Unix domain datagram sockets, start listening for incoming datagrams on a
socket specified by `path`. Note that clients may `send()` without `bind()`,
but no datagrams will be received without a `bind()`.
UNIX 도메인 데이타그램 소켓용. path로 명시된 소켓에서 데이터그램의 수신대기를 시작한다.
클라이언트는 bind()없이 send() 할수 있지만, bind() 없이 데이타그램을 수실하진 않는다.

Example of a Unix domain datagram server that echoes back all messages it receives:
수신한 모든 메세지를 반향하는 Unix 도메인 데이타그램의 예제.

    var dgram = require("dgram");
    var serverPath = "/tmp/dgram_server_sock";
    var server = dgram.createSocket("unix_dgram");

    server.on("message", function (msg, rinfo) {
      console.log("got: " + msg + " from " + rinfo.address);
      server.send(msg, 0, msg.length, rinfo.address);
    });

    server.on("listening", function () {
      console.log("server listening " + server.address().address);
    })

    server.bind(serverPath);

Example of a Unix domain datagram client that talks to this server:
이 서버와 통신하는 UNIX 도메인 데이타그램 클라이언트 예제.

    var dgram = require("dgram");
    var serverPath = "/tmp/dgram_server_sock";
    var clientPath = "/tmp/dgram_client_sock";

    var message = new Buffer("A message at " + (new Date()));

    var client = dgram.createSocket("unix_dgram");

    client.on("message", function (msg, rinfo) {
      console.log("got: " + msg + " from " + rinfo.address);
    });

    client.on("listening", function () {
      console.log("client listening " + client.address().address);
      client.send(message, 0, message.length, serverPath);
    });

    client.bind(clientPath);

### dgram.bind(port, [address])

For UDP sockets, listen for datagrams on a named `port` and optional `address`.  If
`address` is not specified, the OS will try to listen on all addresses.
UDP 소켓용. 명명된 port과 옵셩인 address 데이터 그램을 수신대기 한다. address가 지정되지
않으면 OS 는 모두 address에서 수신을 시도할 것이다.

Example of a UDP server listening on port 41234:
41234 포트에서 수신 대기하는 UDP 서버 예제.

    var dgram = require("dgram");

    var server = dgram.createSocket("udp4");
    var messageToSend = new Buffer("A message to send");

    server.on("message", function (msg, rinfo) {
      console.log("server got: " + msg + " from " +
        rinfo.address + ":" + rinfo.port);
    });

    server.on("listening", function () {
      var address = server.address();
      console.log("server listening " +
          address.address + ":" + address.port);
    });

    server.bind(41234);
    // server listening 0.0.0.0:41234


### dgram.close()

Close the underlying socket and stop listening for data on it.  UDP sockets 
automatically listen for messages, even if they did not call `bind()`.
하층 소켓을 닫고 여기서 데이터의 수시대기를 중지한다. 비록 bind() 가 호출되지 않더라도,
UDP 소켓은 자동으로 메세지를 기다린다.

### dgram.address()

Returns an object containing the address information for a socket.  For UDP sockets, 
this object will contain `address` and `port`.  For Unix domain sockets, it will contain
only `address`.
소켓의 주소 정보를 가지고 있는 객체를 반환한다. UDP 소켓에서 이 객체는 address와 port를 가지고 있다.
UNIX 도메인 소켓에서는 address만 가지고 있다.

### dgram.setBroadcast(flag)

Sets or clears the `SO_BROADCAST` socket option.  When this option is set, UDP packets
may be sent to a local interface's broadcast address.
소켓 옵션 SO_BROADCAST을 설정하거나 해제합니다(복수개). 이 옵션을 세팅하면, UDP 패킷은 로컬
인터페이스의 브로드 캐스트를위한 주소로 발신된다.

### dgram.setTTL(ttl)

Sets the `IP_TTL` socket option.  TTL stands for "Time to Live," but in this context it
specifies the number of IP hops that a packet is allowed to go through.  Each router or 
gateway that forwards a packet decrements the TTL.  If the TTL is decremented to 0 by a
router, it will not be forwarded.  Changing TTL values is typically done for network 
probes or when multicasting.
소켓 옵션의 IP_TTL을 설정한다. TTL은 "수명"을 나타내지만, 이 컨텍스트에서는 패킷 통과를 허용하는
IP 홉 수를 지정한다. 각 라우터 또는 게이트웨이 패킷을 전달하기 위해 TTL을 감소시킨다. 라우터에서
TTL 이 0으로 감소하면, 그것은 전달되지 않는다. TTL 값변경은 일반적으로 네트워크 조사와 멀티
캐스트로 사용된다.

The argument to `setTTL()` is a number of hops between 1 and 255.  The default on most
systems is 64.
setTTL() 의 인자는 1~255의 홉 수이다. 기본값은 대부분 시스템이 64이다.


## Assert

This module is used for writing unit tests for your applications, you can
access it with `require('assert')`.
이 모듈은 응용 프로그램의 단위 테스트를 작성하는데 사용되고 `require('assert')`으로
접근할 수 있다.

### assert.fail(actual, expected, message, operator)

Tests if `actual` is equal to `expected` using the operator provided.
`actual`이 `expected` 와 동일하거나 제공한 연산자를 사용하여 테스트한다.

### assert.ok(value, [message])

Tests if value is a `true` value, it is equivalent to `assert.equal(true, value, message);`
value가 `true`인지 테스트 하고, `assert.equal(true, value, message);` 와 같다.

### assert.equal(actual, expected, [message])

Tests shallow, coercive equality with the equal comparison operator ( `==` ).
'==' 연산자를 적용하여 얕은 등가성을 테스트 한다.

### assert.notEqual(actual, expected, [message])

Tests shallow, coercive non-equality with the not equal comparison operator ( `!=` ).
`!=` 연산자를 적용하여 얕은 비 등가성을 테스트 한다.

### assert.deepEqual(actual, expected, [message])

Tests for deep equality.
깊은 등가성을 테스트 한다.

### assert.notDeepEqual(actual, expected, [message])

Tests for any deep inequality.
깊은 비 등가성을 테스트 한다.

### assert.strictEqual(actual, expected, [message])

Tests strict equality, as determined by the strict equality operator ( `===` )
`===` 연산자 정확한 등가성을 테스트 한다.

### assert.notStrictEqual(actual, expected, [message])

Tests strict non-equality, as determined by the strict not equal operator ( `!==` )
`!==` 연산자 정확한 비 등가성을 테스트 한다.

### assert.throws(block, [error], [message])

Expects `block` to throw an error.
`block` 이 오류가 발생하는 것을 기대한다.

### assert.doesNotThrow(block, [error], [message])

Expects `block` not to throw an error.
`block` 이 오류가 발생하지 않는 것을 기대한다.

### assert.ifError(value)

Tests if value is not a false value, throws if it is a true value. Useful when testing the first argument, `error` in callbacks.
value가 false 인 것을 테스트 하고 true인 경우에 throw. 콜백의 첫번째 인자가 `error`인지 테스팅할 때 유용하다.

## Path

This module contains utilities for dealing with file paths.  Use
`require('path')` to use it.  It provides the following methods:
이 모듈은 파일 경로를 취급하는 유용한 기능들을 포함한다.  사용하려면
`require('path')`를 호출해야 한다.  이 모듈은 다음의 메소드들을 제공한다.

### path.join([path1], [path2], [...])

Join all arguments together and resolve the resulting path.
모든 인수를 하나로 묶고 그 결과로 경로를 결정한다.

Example:
예:

    node> require('path').join(
    ...   '/foo', 'bar', 'baz/asdf', 'quux', '..')
    '/foo/bar/baz/asdf'

### path.normalizeArray(arr)

Normalize an array of path parts, taking care of `'..'` and `'.'` parts.
경로 요소의 배열을 정규화한다.  `'..'` 와 `'.'` 요소를 주의해야 한다.

Example:
예:

    path.normalizeArray(['', 
      'foo', 'bar', 'baz', 'asdf', 'quux', '..'])
    // returns
    [ '', 'foo', 'bar', 'baz', 'asdf' ]

### path.normalize(p)

Normalize a string path, taking care of `'..'` and `'.'` parts.
문자열 경로를 정규화 한다.  `'..'`와 `'.'` 요소를 주의해야 한다.

Example:

    path.normalize('/foo/bar/baz/asdf/quux/..')
    // returns
    '/foo/bar/baz/asdf'

### path.dirname(p)

Return the directory name of a path.  Similar to the Unix `dirname` command.
경로에 있는 디렉토리 이름을 반환한다. Unix의 `dirname` 명령과 유사하다.

Example:
예:

    path.dirname('/foo/bar/baz/asdf/quux')
    // returns
    '/foo/bar/baz/asdf'

### path.basename(p, [ext])

Return the last portion of a path.  Similar to the Unix `basename` command.
경로의 마지막 요소를 반환한다.  Unix의 `basename` 명령과 유사하다.

Example:
예:

    path.basename('/foo/bar/baz/asdf/quux.html')
    // returns
    'quux.html'

    path.basename('/foo/bar/baz/asdf/quux.html', '.html')
    // returns
    'quux'

### path.extname(p)

Return the extension of the path.  Everything after the last '.' in the last portion
of the path.  If there is no '.' in the last portion of the path or the only '.' is
the first character, then it returns an empty string.  Examples:
경로의 확장명을 반환한다.  경로의 마지막 요소에 마지막 '.' 뒤에 문자열이다.  마지막 요소에 '.'이
포함되지 않는 경우, 혹은 '.'이 첫 글자이면 빈 문자열을 반환한다. 예:

    path.extname('index.html')
    // returns 
    '.html'

    path.extname('index')
    // returns
    ''

### path.exists(p, [callback])

Test whether or not the given path exists.  Then, call the `callback` argument with either true or false.  Example:
주어진 경로가 존재하는지 검사한다.  그리고 인수 `callback`을 true 혹은 false의 결과로 호출한다. 예:

    path.exists('/etc/passwd', function (exists) {
      sys.debug(exists ? "it's there" : "no passwd!");
    });


## URL

This module has utilities for URL resolution and parsing.
Call `require('url')` to use it.
이 모듈은 URL 확인 및 분석을 위한 유용한 기능들을 가졌다.
사용하려면 `require('url')` 를 호출해야 한다.

Parsed URL objects have some or all of the following fields, depending on
whether or not they exist in the URL string. Any parts that are not in the URL
string will not be in the parsed object. Examples are shown for the URL
분석된 URL 객체는 URL 문자열에 존재 여부에 따라 다음과 같은 필드를 일부 또는 전체를
갖는다.  URL 문자열에 포함되지 않는 필드는 분석 결과 객체에 포함되지 않는다.
다음과 같은 URL이 있다.

`'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

- `href`

  The full URL that was originally parsed. Example:
  `'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`
  분석하기 전의 전체 URL 이다. 예:
  `'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

- `protocol`

  The request protocol.  Example: `'http:'`
  요청 프로토콜. 예: `'http:'`

- `host`

  The full host portion of the URL, including port and authentication information. Example:
  `'user:pass@host.com:8080'`
  URL 전체 호스트 정보. 인증 정보를 포함한다. 예: `'user:pass@host.com:8080'`

- `auth`

  The authentication information portion of a URL.  Example: `'user:pass'`
  URL 인증정보. 예: `'user:pass'`

- `hostname`

  Just the hostname portion of the host.  Example: `'host.com'`
  호스트 정보 중에 호스트 이름. 예:`'host.com'`

- `port`

  The port number portion of the host.  Example: `'8080'`
  호스트 정보 중에 포트번호이다. 예: `'8080'`

- `pathname`

  The path section of the URL, that comes after the host and before the query, including the initial slash if present.  Example: `'/p/a/t/h'`
  URL의 경로 부분. 호스트 정보에서 검색어 사이에 위치하고 있으며, 처음에 슬래시가 있으면 그것도 포함된다. 예: `'/p/a/t/h'`

- `search`

  The 'query string' portion of the URL, including the leading question mark. Example: `'?query=string'`
  URL 의 `query string` 이고 첫 물음표도 포함된다. 예: `'?query=string'`

- `query`

  Either the 'params' portion of the query string, or a querystring-parsed object. Example:
  `'query=string'` or `{'query':'string'}`
  쿼리 변수 부분의 문자열이거나 쿼리 문자열 구문을 구분하는 개체이다. 예

- `hash`

  The 'fragment' portion of the URL including the pound-sign. Example: `'#hash'`
  URL의 # 표시를 포함하는 부분. 예: `'#hash'`


The following methods are provided by the URL module:
다음 메서드드은 URL 모듈에서 제공된다.

### url.parse(urlStr, parseQueryString=false)

Take a URL string, and return an object.  Pass `true` as the second argument to also parse
the query string using the `querystring` module.
URL 문자열을 인수로 해석한 결과를 반환한다. `querystring` 모듈을 이용하여 쿼리 문자열을 구문 분석하려면
두번째 인수에 `true`를 전달한다.

### url.format(urlObj)

Take a parsed URL object, and return a formatted URL string.
URL 객체를 인수로 포맷화하는 URL 문자열을 반환한다.

### url.resolve(from, to)

Take a base URL, and a href URL, and resolve them as a browser would for an anchor tag.
기본이 되는 URL과 상대 URL을 브러우저에서 앵커 태그가 수행하는 것과 같이 URL을 확인한다.


## Query String(쿼리 스트링)

This module provides utilities for dealing with query strings.  It provides the following methods:
이 모듈은 쿼리 스트링을 다루기 위한 유용한 것들을 제공한다.  다음과 같은 메소드들을 제공한다.

### querystring.stringify(obj, sep='&', eq='=', munge=true)

Serialize an object to a query string.  Optionally override the default separator and assignment characters.
쿼리 객체를 문자열로 직렬화한다. 선택적으로 기본 구분 문자를 오버라이드 한다.
Example:
예:

    querystring.stringify({foo: 'bar'})
    // returns
    'foo=bar'

    querystring.stringify({foo: 'bar', baz: 'bob'}, ';', ':')
    // returns
    'foo:bar;baz:bob'

By default, this function will perform PHP/Rails-style parameter munging for arrays and objects used as
values within `obj`.
이 함수는 기본적으로 PHP/Rails 스타일과 같이 매개변수로 복잡한 작업을 수행한다. 배열이나 개체에 `obj`에 포함된 값을
사용하여 작업을 수행한다.
Example:
예:

    querystring.stringify({foo: ['bar', 'baz', 'boz']})
    // returns
    'foo%5B%5D=bar&foo%5B%5D=baz&foo%5B%5D=boz'

    querystring.stringify({foo: {bar: 'baz'}})
    // returns
    'foo%5Bbar%5D=baz'

If you wish to disable the array munging (e.g. when generating parameters for a Java servlet), you
can set the `munge` argument to `false`.
Example:
만약 배열에 대한 복잡한 처리를 해제하려면(Java 서블릿에 매개변수를 생성하는 경우등) 인수 `munge` 대해 false를
설정할 수 있다. 예:

    querystring.stringify({foo: ['bar', 'baz', 'boz']}, '&', '=', false)
    // returns
    'foo=bar&foo=baz&foo=boz'

Note that when `munge` is `false`, parameter names with object values will still be munged.
`munge`가 `false`일 때에도 값으로 객체가 설정되어 있다면 복잡하게 처리 될 수 있다는 것을 주의해야 한다.

### querystring.parse(str, sep='&', eq='=')

Deserialize a query string to an object.  Optionally override the default separator and assignment characters.
쿼리 문자열을 객체로 복원한다.  선택적으로 기본 구분 문자를 오버라이드 한다.


    querystring.parse('a=b&b=c')
    // returns
    { 'a': 'b'
    , 'b': 'c'
    }

This function can parse both munged and unmunged query strings (see `stringify` for details).
이 함수는 복잡화된 쿼리 문자열과 그렇지 않는 경우 모두 분석할 수 있다. (자세한 내용은 `stringify` 참조하세요.)

### querystring.escape

The escape function used by `querystring.stringify`, provided so that it could be overridden if necessary.
escape 함수는 `querystring.stringify`에서 사용되고 있으며  필요에 따라 재정의 할 수 있도록 제공한다.

### querystring.unescape

The unescape function used by `querystring.parse`, provided so that it could be overridden if necessary.
unescape 함수는 `querystring.parse` 에서 사용되고 있으며 필요에 따라 재정의 할 수 있도록 제공한다.


## REPL

A Read-Eval-Print-Loop (REPL) is available both as a standalone program and easily
includable in other programs.  REPL provides a way to interactively run
JavaScript and see the results.  It can be used for debugging, testing, or
just trying things out.
Read-Eval-Print-Loop (PEPL)은 독립실행형 프로그램과 손쉽게 다른 프로그램을 통합하여 이용할
수 있다.  REPL 은 인터렉티브하게 JavaScript를 실행하고 결과를 볼 수 있는 방법을 제공한다.
이것은 디버깅, 테스팅 혹은 기타 여러가지를 시도하는 용도로 사용된다.

By executing `node` without any arguments from the command-line you will be
dropped into the REPL. It has simplistic emacs line-editing.
명렬줄에서 인수 없이 `node`를 실행하면 PEPL 프로그램 시스템으로 진입한다.  PEPL은
단순한 Emacs 라인 에디터이다.

    mjr:~$ node
    Type '.help' for options.
    node> a = [ 1, 2, 3];
    [ 1, 2, 3 ]
    node> a.forEach(function (v) {
    ...   console.log(v);
    ...   });
    1
    2
    3

For advanced line-editors, start node with the environmental variable `NODE_NO_READLINE=1`.
This will start the REPL in canonical terminal settings which will allow you to use with `rlwrap`.
더욱 진보된 라인 에디터를 위해서 환경변수에 `NODE_NO_READLINE=1`를 설정하고 node를 실행한다.
따라서 표준 터미널 설정의 REPL 을 실행하고 `rlwrap` 사용 상태에서 REPL을 이용할 수 있다.
                                     
For example, you could add this to your bashrc file:
예를 들어, bashrc 파일에 다음과 같이 설정을 추가한다.

    alias node="env NODE_NO_READLINE=1 rlwrap node"


### repl.start(prompt='node> ', stream=process.openStdin())

Starts a REPL with `prompt` as the prompt and `stream` for all I/O.  `prompt`
is optional and defaults to `node> `.  `stream` is optional and defaults to 
`process.openStdin()`.
프롬프트 기호와 같은 `prompt`와 모든 I/O `stream`을 가지고 시작한다. `prompt`는 생략
가능하며 기본값은 `node> `입니다.  `stream`은 선택사항이고 기본값은
`process.openStdin()`이다.

Multiple REPLs may be started against the same running instance of node.  Each
will share the same global object but will have unique I/O.
여러 REPL을 실행하면 동일한 node 인스턴스가 실행되지 않을 수 있다. 각 REPL은 전역 객체
를 공유하지만 유일한 I/O를 갖는다.

Here is an example that starts a REPL on stdin, a Unix socket, and a TCP socket:
REPL 표준 입력, Unix 도메인 소켓, TCP 소켓하에 시작한다.

    var net = require("net"),
        repl = require("repl");

    connections = 0;

    repl.start("node via stdin> ");

    net.createServer(function (socket) {
      connections += 1;
      repl.start("node via Unix socket> ", socket);
    }).listen("/tmp/node-repl-sock");

    net.createServer(function (socket) {
      connections += 1;
      repl.start("node via TCP socket> ", socket);
    }).listen(5001);

Running this program from the command line will start a REPL on stdin.  Other
REPL clients may connect through the Unix socket or TCP socket. `telnet` is useful
for connecting to TCP sockets, and `socat` can be used to connect to both Unix and
TCP sockets.
이 프로그램은 명령줄에서 실행하면 표준 입력에 따라 REPL이 시작된다. 다른 REPL 클라이언트는
UNIX 도메인 소켓 또는 TCP 소켓을 통해 연결할 수 있다. `telnet`은 TCP 소켓 연결에 유용하다.
`socket`은 UNIX와 TCP 소켓에 연결하기 위해 사용할 수 있다.

By starting a REPL from a Unix socket-based server instead of stdin, you can 
connect to a long-running node process without restarting it.
표준 입력 대신에 UNIX 도메인 소켓을 기반으로 한 서버에서 REPL을 시작하고 다시 시작하지
않고 node 상주 프로세스에 연결할 수 있다.


### REPL Features(REPL의 장점)


Inside the REPL, Control+D will exit.  Multi-line expressions can be input.
REPL에서 Control+D를 하면 종료한다.  여러 줄의 표현식을 입력할 수 있다.

The special variable `_` (underscore) contains the result of the last expression.
특별한 변수 `_` (밑줄)은 맨 마지막 표현식의 결과를 유지한다.

    node> [ "a", "b", "c" ]
    [ 'a', 'b', 'c' ]
    node> _.length 
    3
    node> _ += 1
    4

The REPL provides access to any variables in the global scope. You can expose a variable 
to the REPL explicitly by assigning it to the `context` object associated with each
`REPLServer`.  For example:
REPL은 전역에 존재하는 모든 변수에 접근할 수 있다. 각 `REPLServer`에 관련된 `context` 객체를
부여하여 명시적으로 변수를 노출시킬 수 있다. 예시 :

    // repl_test.js
    var repl = require("repl"),
        msg = "message";

    repl.start().context.m = msg;

Things in the `context` object appear as local within the REPL:
`context` 개체에 설정된 변수는 REPL안에 로컬 변수로 나타날 것이다.

    mjr:~$ node repl_test.js 
    node> m
    'message'

There are a few special REPL commands:
REPL은 몇 가지 특별한 명령이 있다.

  - `.break` - While inputting a multi-line expression, sometimes you get lost or just don't care 
  about completing it.  `.break` will start over.
  - `.break` - 여러 행에 걸쳐 수식을 입력하는 동안 때로 중간에 잃어 버리거나 수식을 완료하지 않더라도
  `.break`는 처음부터 다시 시작한다.
  
  - `.clear` - Resets the `context` object to an empty object and clears any multi-line expression.
  - `.clear` - `context` 객체를 빈 상태로 리셋하고, 여러 줄로 입력한 수식을 지운다.
  
  - `.exit` - Close the I/O stream, which will cause the REPL to exit.
  - `.exit` - I/O 스트림을 닫고 REPL을 종료한다.

  - `.help` - Show this list of special commands.
  - `.help` - 명령 목록을 표시한다.
  

## Modules

Node uses the CommonJS module system.
Node 는 CommonJS 의 모듈 시스템을 사용한다.

Node has a simple module loading system.  In Node, files and modules are in
one-to-one correspondence.  As an example, `foo.js` loads the module
`circle.js` in the same directory.
Node는 간단한 모듈 로딩 시스템이 있다. Node에서 파일과 모듈은 1대 1 대응을 이룬다.
예를 들어 `foo.js` 는 같은 디렉토리에 있는 `circle.js` 모듈을 로드한다.

The contents of `foo.js`:
`foo.js`의 내용

    var circle = require('./circle');
    console.log( 'The area of a circle of radius 4 is '
               + circle.area(4));

The contents of `circle.js`:
`circle.js` 의 내용

    var PI = 3.14;

    exports.area = function (r) {
      return PI * r * r;
    };

    exports.circumference = function (r) {
      return 2 * PI * r;
    };

The module `circle.js` has exported the functions `area()` and
`circumference()`.  To export an object, add to the special `exports`
object.  (Alternatively, one can use `this` instead of `exports`.) Variables
local to the module will be private. In this example the variable `PI` is
private to `circle.js`. The function `puts()` comes from the module `'sys'`,
which is a built-in module. Modules which are not prefixed by `'./'` are
built-in module--more about this later.
`circle.js`는 `area()` 와 `circumference()` 함수를 내보낸다.  개체를 내보내기
위해서는 지정된 `exports` 객체에 추가해야 한다. (선택적으로 exports 대신에 this를
사용할 수 있다.) 모듈의 로컬 변수는 비공개이다. 이 경우 변수 PI는 `circle.js`의
지역변수이다. 함수 `put()`는 빌트인 모듈이다. 자세한 사항은 다음에 설명한다.

A module prefixed with `'./'` is relative to the file calling `require()`.
That is, `circle.js` must be in the same directory as `foo.js` for
`require('./circle')` to find it.
접두사 `'./'` 가 붙는 모듈은 `require()` 를 호출하는 모듈에 대한 상대 경로이다.
즉 `circle.js` 는 `require('./circle')`로 찾을 수 있도록 `foo.js`와 같은 디렉
 토리에 있을 필요가 있다.

Without the leading `'./'`, like `require('assert')` the module is searched
for in the `require.paths` array. `require.paths` on my system looks like
this:
첫번째 `'./'` 없이, 예를 들면 `require('assert')` 와 같이 모듈을 지정하면
`require.paths` 배열의 위치를 기점으로 검색된다. 내(ryan) 시스템에서는 다음과 같다.

`[ '/home/ryan/.node_libraries' ]`

That is, when `require('assert')` is called Node looks for:
따라서 `require('assert')` 이 호출된 경우, Node 는 다음의 순서로 모듈을 찾는다.

* 1: `/home/ryan/.node_libraries/assert.js`
* 2: `/home/ryan/.node_libraries/assert.node`
* 3: `/home/ryan/.node_libraries/assert/index.js`
* 4: `/home/ryan/.node_libraries/assert/index.node`

interrupting once a file is found. Files ending in `'.node'` are binary Addon
Modules; see 'Addons' below. `'index.js'` allows one to package a module as
a directory.
파일이 발견되면 그 시점에서 검색이 종료된다. `'.node'`로 끝나는 파일은 바이너리 부가기능
이다. 자세한 내용은 '추가기능'을 참고하세요. `'index.js'`는 디렉토리를 하나의 모듈로
묶는 것을 허용한다.

`require.paths` can be modified at runtime by simply unshifting new
paths onto it, or at startup with the `NODE_PATH` environmental
variable (which should be a list of paths, colon separated).
`require.paths`는 배열에 새 경로를 추가하거나, NODE_PATH 환경 변수로 시작하여
변경할 수 있다. (이 경우는 클론으로 구분된 경로 목록을 전달 해야 한다.)


## Addons

Addons are dynamically linked shared objects. They can provide glue to C and
C++ libraries. The API (at the moment) is rather complex, involving
knowledge of several libraries:
부가 기능은 동적으로 공유 객체에 연결된다. 그것들은 C, C++ 라이브러리를 위한 연결
지점을 제공한다. API는 일부 라이브러리의 지식을 포함하며, (현재) 매우 복잡하다.

 - V8 JavaScript, a C++ library. Used for interfacing with JavaScript:
   creating objects, calling functions, etc.  Documented mostly in the
   `v8.h` header file (`deps/v8/include/v8.h` in the Node source tree).
 - V8 자바스크립트 엔진은 C++ 라이브러리이다. JavaScript의 오브젝트를 만들거나
   함수 호출등의 인터페이스에 사용된다.  문서화는 대부 v8.h 헤더파일에 기록되분
   어 있다.   (Node 소스 트리의 deps/v8/include/v8.h)

 - libev, C event loop library. Anytime one needs to wait for a file
   descriptor to become readable, wait for a timer, or wait for a signal to
   received one will need to interface with libev.  That is, if you perform
   any I/O, libev will need to be used.  Node uses the `EV_DEFAULT` event
   loop.  Documentation can be found http:/cvs.schmorp.de/libev/ev.html[here].
   libev는 C 이벤트 루프 라이브러리이다. 파일 기술자가 읽을 수 있게되고, 타이머를 기다릴
   때 혹은 신호를 받기를 기다릴 때 등에서 libev 인터페이스가 필요하다. 그것은 즉 어떤 I/O
   작업을 하려면 반드시 libev 을 써야할 필요가 있다는 것이다. Node는 `EV_DEFAULT` 이벤트
   루프를 사용한다.  문서는 http:/cvs.schmorp.de/libev/ev.html [here] 에 있다.

 - libeio, C thread pool library. Used to execute blocking POSIX system
   calls asynchronously. Mostly wrappers already exist for such calls, in
   `src/file.cc` so you will probably not need to use it. If you do need it,
   look at the header file `deps/libeio/eio.h`.
   libeio는 C 쓰레드 풀 라이브러리이다.  블럭킹 POSIX 시스템을 비동기적으로 실행하
   는데 사용한다.  이러한 호출을 위한 대부분의 래퍼는 이미 src/file.cc에 포함되어
   있기 때문에, 아마 그것을 사용하게 될 것이다.  필요한 경우, deps/libeio/eio.h
   헤더 파일을 참조하세요.

 - Internal Node libraries. Most importantly is the `node::ObjectWrap`
   class which you will likely want to derive from.
   Node 내부 라이브러리에서 가장 중요한 것은 `node::ObjectWrap` 클래스이다.

 - Others. Look in `deps/` for what else is available.
   ?다른 것들은 여부는 `deps/` 다음을 참조하세요.

Node statically compiles all its dependencies into the executable. When
compiling your module, you don't need to worry about linking to any of these
libraries.
Node는 런타임에 의존하는 소스를 정적으로 컴파일한다. 모듈을 컴파일 할때 여기 라이브러리
들 모두는 그 연결에 대하여 아무런 주의도 필요로 하지 않는다.

To get started let's make a small Addon which does the following except in
C++:
먼저 작은 부가기능을 만들어 보기 위해서 C++에서 다음과 같이 동작하는 작은 기능을 만들어 보세요.

    exports.hello = 'world';

To get started we create a file `hello.cc`:
먼저 hello.cc 라는 파일을 만듭니다.

    #include <v8.h>

    using namespace v8;

    extern "C" void
    init (Handle<Object> target) 
    {
      HandleScope scope;
      target->Set(String::New("hello"), String::New("World"));
    }

This source code needs to be built into `hello.node`, the binary Addon. To
do this we create a file called `wscript` which is python code and looks
like this:
이 소스 코드는 `hello.node`에 바이너리 부가기능 빌드가 필요하다.  빌드를 하기 위해서
는 `wscript` 라고 하는 다음과 같은 파이썬 코드를 생성한다.

    srcdir = '.'
    blddir = 'build'
    VERSION = '0.0.1'

    def set_options(opt):
      opt.tool_options('compiler_cxx')

    def configure(conf):
      conf.check_tool('compiler_cxx')
      conf.check_tool('node_addon')

    def build(bld):
      obj = bld.new_task_gen('cxx', 'shlib', 'node_addon')
      obj.target = 'hello'
      obj.source = 'hello.cc'

Running `node-waf configure build` will create a file
`build/default/hello.node` which is our Addon.
`node-qaf configurae build`를 실행하면 `build/default/hello.node` 부가기능
파일이 만들어 진다.  

`node-waf` is just http://code.google.com/p/waf/[WAF], the python-based build system. `node-waf` is
provided for the ease of users.
`node-waf` 는 http://code.google.com/p/waf/[WAF]에 있고 Python 기반의 빌드이다.
`node-waf` 는 사용자의 부담을 줄이기 위해서 제공되고 있다.

All Node addons must export a function called `init` with this signature:
Node의 모든 부가 기능들은 이 서명과 함께 `init` 함수 호출되도록 내보내야만 한다.

    extern 'C' void init (Handle<Object> target)

For the moment, that is all the documentation on addons. Please see
<http://github.com/ry/node_postgres> for a real example.
현재 추가된 문서는 이게 전부이고 실제 예제는 http://github.com/ry/node_postgres 를
참조하세요.


## Appendix - Third Party Modules

There are many third party modules for Node. At the time of writing, August
2010, the master repository of modules is
http://github.com/ry/node/wiki/modules[the wiki page].
Node 를 위한 수 많은 써드 파티 모듈들이 있다. 이 글을 쓰는 시점인 2010년 8월에 모듈
master 저장소는 http://github.com/ry/node/wiki/modules[the wiki page].

This appendix is intended as a SMALL guide to new-comers to help them
quickly find what are considered to be quality modules. It is not intended
to be a complete list.  There may be better more complete modules found
elsewhere.
이 부록은 괜찮은 모듈이라고 생각하는 모듈을 초보자가 빠르게 찾을 수 있도록 도와주는
작은 가이드 정도라고 생각하면 된다. 이것은 완벽한 리스트가 아니며 다른 곳에서 더
완벽한 모듈을 찾을지 모른다.

- Module Installer: [npm](http://github.com/isaacs/npm)

- HTTP Middleware: [Connect](http://github.com/senchalabs/connect)

- Web Framework: [Express](http://github.com/visionmedia/express)

- Web Sockets: [Socket.IO](http://github.com/LearnBoost/Socket.IO-node)

- HTML Parsing: [HTML5](http://github.com/aredridel/html5)

- [mDNS/Zeroconf/Bonjour](http://github.com/agnat/node_mdns)

- [RabbitMQ, AMQP](http://github.com/ry/node-amqp)

- [mysql](http://github.com/felixge/node-mysql)

- Serialization: [msgpack](http://github.com/pgriess/node-msgpack)

- Scraping: [Apricot](http://github.com/silentrob/Apricot)

- Debugger: [ndb](http://github.com/smtlaissezfaire/ndb) is a CLI debugger
  [inspector](http://github.com/dannycoates/node-inspector) is a web based
  tool.

- [pcap binding](http://github.com/mranney/node_pcap)

- [ncurses](http://github.com/mscdex/node-ncurses)

- Testing/TDD/BDD: [vows](http://vowsjs.org/),
  [expresso](http://github.com/visionmedia/expresso),
  [mjsunit.runner](http://github.com/tmpvar/mjsunit.runner)

Patches to this list are welcome.
이 목록에 패치를 해주는 것은 언제든지 환영한다.
