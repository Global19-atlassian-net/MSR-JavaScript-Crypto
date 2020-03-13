﻿## Overview

The Microsoft Research JavaScript Cryptography Library has been developed for use with cloud services in an HTML5 compliant and forward-looking manner. The algorithms are exposed via the [W3C Web Cryptography API](https://www.w3.org/TR/WebCryptoAPI/)

The library currently supports RSA encrypt (OAEP) and digital signature (PSS); AES-CBC and GCM encrypt/decrypt; SHA-256/384/512, HMAC with supported hash functions; PRNG (AES-CTR based) as specified by NIST; ECDH; ECDSA; and KDF (Concat). The library is tested on IE8,9,10,11, Microsoft Edge and latest Firefox, Chrome, Opera, and Safari browsers.    
This library includes big-number integer arithmetic to support the aforementioned cryptographic algorithms. It supports unsigned big integer arithmetic with addition, subtraction, multiplication, division, reduction, inversion, GCD, extended Euclidean algorithm (EEA), Montgomery multiplication, and modular exponentiation. It provides useful utility functions, such as endianness management and conversion routines. The big integer library is likely to change in future releases. There are also unit tests and some sample code. Future updates to this library may change the programming interfaces.    

---
## Recommended Usage
It is recommended that this library be used as a polyfill for the native Web Crypto API supported in modern browsers.
It is strongly advised that you used the native browser crypto whenever available. It will be more thoroughly tested, more secure, and have significantly better performance.  
To select native crypto, when available, you can use the following code:  
```
var crypto = window.msCrypto /*IE11*/ || window.crypto /*native*/ || window.msrCrypto;
crypto.subtle.encrypt(...);
```
It is not recommended to use this library in a server type application.

---
## Supported Algorithms

Encryption/Decryption:  
- RSA-OAEP
- AES-GCM
- AES-CBC (_no longer recommended. Use AES-GCM. We continue support for compatibility_)

Digital Signature  
- RSA-PSS 
- RSA-PKCSv1.15
- HMAC
- ECDSA

Hash  
- SHA-1 (_no longer recommended. Use SHA-2. We continue support for compatibility_)
- SHA-224
- SHA-256
- SHA-384
- SHA-512

Derive Key/Bits  
- ECDH
- Concat-KDF

KeyWrap  
- AES-GCM

Supported ECC curves:  
- P-256, P-384, P-521, BN-254, NUMSP256D1, NUMSP256T1, NUMSP384D1, NUMSP384T1

----
## Building the library
>There are pre-built library files in the lib folder. `/lib/msrCrypto.js` and `/lib/msrCrypto.min.js`  

>While this library has some npm build dependencies, it has no runtime dependencies.

You may build the library from the source files. The library is built using _gulp_ from npm to concatenate many individual JavaScript files into a single library file.  
Run `npm install` from a command terminal to install the required _npm_ packages. `gulpfile.js` contains a list of scripts included in the build. You may remove scripts to create a subset of the library that supports fewer algorithms. Be aware, many scripts have dependencies on other scripts to function properly.

### Building from VS Code:
>_This assumes the npm packages have been installed (`>npm install`) and that the workspace file (msrcrypto.code-workspace) was used to load the project._  

Select the _Terminal_ menu; Select _Run Build Task..._ to build a library file in `/lib`  
Alternately you can use the _ctrl+shift+b_ keyboard shortcut.  
Or from the 'Command Palette...' select 'Tasks: Run Build Task'  
Or from the Terminal window: `.\node_modules\.bin\gulp`

----
## Additional Utilities
msrCrypto supplies a few data conversion functions that are not part of the Web Cryptography API spec.  
`.textToBytes(String)` converts a string to an array of UTF-8 encoded bytes.  
`.bytesToText(Array|ArrayBuffer|TypedArray)` converts UTF-8 bytes into a string.  
`.toBase64(Array|ArrayBuffer|TypedArray)` converts byte data to a base64 string.  
`.fromBase64(String)` converts base64 string into Array of bytes.  

---
## Limitations and Security    

**Native crypto**   
Developers should always use native platform crypto when available. Native crypto will have improved performance and offer additional security and memory protection not available in JavaScript. Modern browsers support the Web Crypto API.

**Secret/private key data**    
Secret key data is stored in JavaScript's memory and is potentially accessible to other scripts, applications, browser extensions, and developer tools. While key data may be stored outside of JavaScripts memory, the key data will be required in-memory by the algorithms running in JavaScript.

**Side-channel protection**    
We have taken steps to prevent side-channel vulnerabilities where practical in the library. This library is meant to run on many platforms with many JavaScript engines and we cannot control the side-channel prevention measures employed in either the engines or the underlying platforms.    

**HTTPS**    
When this library is used as part of an html application, HTTPS should always be used when connecting to the server. HTTPS will allow your secret keys and data to be transmitted to client across a secure connection.

---
## Browser compatibility

**msrCrypto.js** is compatible with IE8 and newer browsers; latest versions of Chrome, Safari, Opera.  
Browser web crypto uses Typed-Arrays for input and output of data. msrCrypto can use either Typed-Arrays or regular Arrays.

Known issues:  
### IE8:
"Catch" is a reserved keyword, so using Promises.catch() function will throw an error.  
To use the `catch` function use the `promise["catch"]()` form.

### IE8/9:	
No Typed Array Support (`ArrayBuffer`, `UInt8Array`, etc...).  
You must use regular Arrays for inputting data into msrCrypto when using IE8/9.  
Results will be returned as regular Arrays as well. For IE10 and newer, results will be returned as an ArrayBuffer.  
A workaround is to create a simple Uint8Array wrapper for an Array when Typed Arrays are not supported. You wont' get any actual Uint8Array functionality, but you can pass arguments without having to have special cases for your code.
```
if (!Uint8Array) {
	var Uint8Array = function (array) {
		return array;
	}
}

var data = new Uint8Array(dataArray);
```

### IE11
IE11 supports the Web Crypto API, but was based on a pre-release version of the spec and was never updated. So it uses event based calls instead of Promises and a few other quirks of the API.
In the `/lib` folder there is a `IE11PromiseWrapper.js` file. This shim can be loaded in IE11 and allow you to call the native Web Crypto API using the Promise based calling scheme. This shim also corrects some of the quirks in the IE11 API.

---
## Random number generator (PRNG):

Many of msrCrypto's crypto algorithms require random numbers. Random numbers for cryptography
need to be obtained from a cryptographically secure random number generator. This is not 
available on older browsers (IE10, IE9, & IE8). 

msrCrypto has its own secure random number generator written in JavaScript (PRNG). However, the PRNG 
needs to be initialized with some bytes of random entropy that it cannot obtain on older browsers. It is important that this entropy is obtained from a secure random source - such as from a crypto api on the server and sent with the web page over a secure HTTPS connection.

Newer browsers use `Crypto.getRandomValues()` to obtain cryptographically strong random values. Our PRNG will seed itself with this when available.

Once the entropy is obtained initialize the PRNG before calling any functions. 48 random bytes is recommeneded.
```
msrCrypto.initPrng(<randomArrayOf48Bytes>);
```
---
## API Documentation

Since this library uses the standard Web Cryptography API we used to recommend using the official Microsoft documentation for the Microsoft Edge browser. I can no longer find that documentation.  
A good source for documentation is:  
[MDN Web Docs Subtle Crypto](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto)  
[W3C Web Cryptography API](https://www.w3.org/TR/WebCryptoAPI/)

>msrCrypto uses identical calls as these documents with the addition of allowing regular JavaScript Arrays in place of Typed-Arrays.

---
## Updates

Changes with version 1.6.0

	Includes additional side-channel protection.
	Automatic web-worker usage is disabled by default. It was causing too many problems when the library was bundled with other scripts.
	'raw' key import support for HMAC & ECDH.
	.wrapKey support for AES-CBC, AES-GCM, RSA-OAEP.
	Moved to GitHub.

Changes with version 1.5

	Added support for streaming input/output data to crypto calls.
	See Samples/StreamSample.html for an example on how to use this feature.

	Now allow concurrent crypto calls of the same type at the same time. Before, concurrent 
	crypto operations that shared code would possibly return incorrect results.  
	Now, for example, you could perform multiple encryptions at the same time with streaming.

	Added 'raw' keyImport/keyExport format for hmac, AES-CBC, AES-GCM.

	Added IE11PromiseWrapper.js script to wrap the IE11 non-standard WebCrypto api and make
	it function the same as current standard WebCrypto api. Your WebCrypto code should now 
	work with msrCrypto, IE11-WebCrypto, and the current standard WebCrypto with minimal
	special case code.

	Removed rsaes-pkcs1-v1_5 encrypt/decrypt (considered less secure and obsolete)
	It is not supported by WebCrypto in modern browsers.

	Added TypeScript d.ts file.   msrCrypto.d.ts for using with type script.

	Moved the Promise polyfill outside of the msrCrypto library so you can use the built-in
	browser version when available.	 

### Changes with version 1.4

The API has been updated to support the latest Web Crypto Api spec and be compatible with the
implementation on the latest browsers.

Promises are now supported and the IE11 based events are removed. Crypto calls are now in the 
form:
```
	// NEW STYLE with Promises
        msrCrypto.subtle.encrypt(<parameters>).then(
			function(encryptionResult) {

				//... do something here with the result

			},
			function(error) {

				//... handle error

			}
        );
```

This will break code that uses the pre-1.4 calling conventions:
```
	// OLD STYLE with events (before version 1.4)
	var cryptoOperation =  msrCrypto.subtle.encrypt(<parameters>);
	
	cryptoOperation.onComplete = 
	    function(encryptionResult) {

			//... do something here with the result

	    };

	cryptoOperation.onError = 
	    function(encryptionResult) {

			//... handle error

	    };