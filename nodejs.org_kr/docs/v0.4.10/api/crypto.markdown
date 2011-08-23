## Crypto

Use `require('crypto')` to access this module.
이 모듈에 액세스하려면 `require('crypto')` 를 사용한다.

The crypto module requires OpenSSL to be available on the underlying platform.
It offers a way of encapsulating secure credentials to be used as part
of a secure HTTPS net or http connection.
암호화 모듈은 하부의 플랫폼에서 OpenSSL 을 사용할 것을 요구한다. 그것은 안전한 HTTPS
네트워크와 http 연결의 일부로 사용되는 안전한 인증 정보를 캡슐화하는 방법을 제공한다.

It also offers a set of wrappers for OpenSSL's hash, hmac, cipher, decipher, sign and verify methods.
동시에 OpenSSL 해시, HMAC, 암호화, 복호화 서명 그리고 검증에 래퍼를 일체 제공한다.

### crypto.createCredentials(details)

Creates a credentials object, with the optional details being a dictionary with keys:
인증 정보 개체를 만든다. 옵션 details 다음 키가 있는 사전이다.

* `key` : a string holding the PEM encoded private key
`key` : PEM 인코딩된 비밀키를 포함하는 문자열
* `cert` : a string holding the PEM encoded certificate
`cert` : PEM 인코딩된 인증서를 포함하는 문자열
* `ca` : either a string or list of strings of PEM encoded CA certificates to trust.
`ca` : 신뢰할 수 있는 인증 기관의 인증서가 PEM으로 인코딩된 문자열 또는 문자열 배열

If no 'ca' details are given, then node.js will use the default publicly trusted list of CAs as given in
<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>.
`ca` 정보가 주어지지 않는 경우 node.js 는 기본적으로 <http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>
에 주어진 신뢰할 수 있는 인증 기관의 공개 목록을 사용한다.


### crypto.createHash(algorithm)

Creates and returns a hash object, a cryptographic hash with the given algorithm
which can be used to generate hash digests.
해시 객체를 생성하여 반환한다. 주어진 알고리즘으로 암호화 해시 함수는 다이제스트를 생성하는데 사용한다.

`algorithm` is dependent on the available algorithms supported by the version
of OpenSSL on the platform. Examples are `'sha1'`, `'md5'`, `'sha256'`, `'sha512'`, etc.
On recent releases, `openssl list-message-digest-algorithms` will display the available digest algorithms.
`alogrithm` 은 플랫폼에서 OpenSSL 버전에서 지원되는 사용 가능한 알고리즘에 의존한다.
예를 들어 sha1, md5, sha256, sha512 등 이다. 최신 릴리즈에서는 `openssl list-message-digest-algorithm`에서
사용할 수 있는 다이제스트 알고리즘이 표시된다.

Example: this program that takes the sha1 sum of a file

    var filename = process.argv[2];
    var crypto = require('crypto');
    var fs = require('fs');

    var shasum = crypto.createHash('sha1');

    var s = fs.ReadStream(filename);
    s.on('data', function(d) {
      shasum.update(d);
    });

    s.on('end', function() {
      var d = shasum.digest('hex');
      console.log(d + '  ' + filename);
    });

### hash.update(data)

Updates the hash content with the given `data`.
This can be called many times with new data as it is streamed.
주어진 `data`에서 해시의 내용을 업데이트 한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우

### hash.digest(encoding='binary')

Calculates the digest of all of the passed data to be hashed.
The `encoding` can be `'hex'`, `'binary'` or `'base64'`.
전달된 모든 데이터가 해시 다이제스트를 계산한다. `encoding`은 'hex', 'binary' 또는 'base64' 중 하나이다.


### crypto.createHmac(algorithm, key)

Creates and returns a hmac object, a cryptographic hmac with the given algorithm and key.
주어진 키와 알고리즘으로 암호화된 hmac 객체를 생성하여 반환한다.

`algorithm` is dependent on the available algorithms supported by OpenSSL - see createHash above.
`key` is the hmac key to be used.
`algorithm` 은 OpenSSL에서 지원하는 알고리즘에 따라 달라진다. - 위에서 언급한 createHash 를 참고하세요.
`key`는 hmac 키를 사용한다.

### hmac.update(data)

Update the hmac content with the given `data`.
This can be called many times with new data as it is streamed.
주어진 `data`로 HMAC 내용을 업데이트 한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우 여러번 호출된다.

### hmac.digest(encoding='binary')

Calculates the digest of all of the passed data to the hmac.
The `encoding` can be `'hex'`, `'binary'` or `'base64'`.
전달된 모든 데이터가 HMAC화된 다이제스트를 계산한다. `encoding`은 `'hex'`, `'birnay'` 또는 `'base64'` 중 하나이다.


### crypto.createCipher(algorithm, key)

Creates and returns a cipher object, with the given algorithm and key.
주어진 알고리즘과 키를 사용하여 암호화 개체를 만들어 반환한다.

`algorithm` is dependent on OpenSSL, examples are `'aes192'`, etc.
On recent releases, `openssl list-cipher-algorithms` will display the available cipher algorithms.
`algorithm`은 OpenSSL에 의존한다. 예를 들어 aes192 등을 말한다.
최신 릴리즈에 `oepnssl list-cipher-algorithms`에서 사용할 수 있는 암호화 알고리즘이 표시된다.

### cipher.update(data, input_encoding='binary', output_encoding='binary')

Updates the cipher with `data`, the encoding of which is given in `input_encoding`
and can be `'utf8'`, `'ascii'` or `'binary'`. The `output_encoding` specifies
the output format of the enciphered data, and can be `'binary'`, `'base64'` or `'hex'`.
`date` 암호를 업데이트 한다. `input_encodind` 에 주어진 인코딩은 `utf8`, `ascii` 혹은 `binary` 이다.
`output_encoding`는 암호화된 데이터의 출력로 형식을 지정하는 것으로, `binary`, `base64`, `hex` 중의 하나이다.

Returns the enciphered contents, and can be called many times with new data as it is streamed.
암호호된 컨텐츠를 반환한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우 여러번 호출된다.

### cipher.final(output_encoding='binary')

Returns any remaining enciphered contents, with `output_encoding` being one of: `'binary'`, `'base64'` or `'hex'`.
암호화된 콘텐츠의 나머지를 반환한다. `output_encoding`는 다음중 하나일 수 있다. `'binary'`, `'ascii'` 또는 `'utf8'`

### crypto.createDecipher(algorithm, key)

Creates and returns a decipher object, with the given algorithm and key.
This is the mirror of the cipher object above.
주어진 알고리즘과 키를 사용하여 암호 개체를 반환한다. 이것은 위에서 언급한 암호 개체의 사본이다.

### decipher.update(data, input_encoding='binary', output_encoding='binary')

Updates the decipher with `data`, which is encoded in `'binary'`, `'base64'` or `'hex'`.
The `output_decoding` specifies in what format to return the deciphered plaintext: `'binary'`, `'ascii'` or `'utf8'`.
`binary`, `base64` 또는 'hex'중 하나로 인코딩 된 암호를 `data`로 업데이트 한다.
`output_decoding`이 암호화된 텍스트 형식을 지정하는 것으로, `'binary'`, `'ascii'` 혹은 `'utf8'`중에 하나이다.


### decipher.final(output_encoding='binary')

Returns any remaining plaintext which is deciphered,
with `output_encoding` being one of: `'binary'`, `'ascii'` or `'utf8'`.
암호화된 텍스트의 나머지를 반환한다. `output_encoding` 은 `'binary'`,`'ascii'` 혹은 `'utf8'`중의 하나이다.


### crypto.createSign(algorithm)

Creates and returns a signing object, with the given algorithm.
On recent OpenSSL releases, `openssl list-public-key-algorithms` will display
the available signing algorithms. Examples are `'RSA-SHA256'`.
주어진 알고리즘으로 서명 객체를 만들어 반환한다. 최근 OpenSSL 버전에서는
`openssl list-puglic-key-algorithms`에서 사용할 수 있는 서명 알고리즘의 목록이 표시된다.
예를 들어 `'RSA-SHA256'`.

### signer.update(data)

Updates the signer object with data.
This can be called many times with new data as it is streamed.
서명 객체를 데이터로 업데이트 한다. 이것은 새로운 데이터를 스트림으로 흘러가는 경우 여러번 호출된다.

### signer.sign(private_key, output_format='binary')

Calculates the signature on all the updated data passed through the signer.
`private_key` is a string containing the PEM encoded private key for signing.
서명 객체에 전달된 모든 갱신 데이터 서명을 계산한다. `private_key`는 PEM 인코딩 된
비밀키를 내용으로 하는 문자열이다.

Returns the signature in `output_format` which can be `'binary'`, `'hex'` or `'base64'`.
`'binary'`, `'hex'` 혹은 `'base64'`중 하나를 지정한 `output_format` 서명을 반환한다.

### crypto.createVerify(algorithm)

Creates and returns a verification object, with the given algorithm.
This is the mirror of the signing object above.
주어진 알고리즘으로 검증 객체를 만들어 반환한다. 이것은 이전의 서명 개체와 똑같은 복사본이다.

### verifier.update(data)

Updates the verifier object with data.
This can be called many times with new data as it is streamed.
유효성 검사 개체를 업데이트 한다. 이것은 새로운 데이터 스트림으로 흘러가는 경우 여러번 호출된다.

### verifier.verify(object, signature, signature_format='binary')

Verifies the signed data by using the `object` and `signature`. `object` is  a
string containing a PEM encoded object, which can be one of RSA public key,
DSA public key, or X.509 certificate. `signature` is the previously calculated
signature for the data, in the `signature_format` which can be `'binary'`,
`'hex'` or `'base64'`.
서명된 데이터를 `object`와 `signature`에서 확인한다. `object`는 RSA 공개키, DSA 공개키
, X.509 인증서 중 하나를 PEM으로 인코딩된 객체이다. `signature` 는 이미 계산된 데이터
서명이 `signature_format`은 `'binary'`, `'hex'`, `'base64'` 중 하나이다.

Returns true or false depending on the validity of the signature for the data and public key.
서명된 데이터와 공개키에 의한 검증 결과에 따라 true 또는 false를 반환합니다.
