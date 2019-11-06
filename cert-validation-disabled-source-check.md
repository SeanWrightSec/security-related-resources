# Certificate Validation Disabled Source Code Checks
This file holds common code which is used to disable certificate validation. By searching for this code in your source code you will hopefully be able to identify code in your systems which has had certificate validation disabled.

## Java
### Standard API

The following code is used to disable CA validation of a certificate:
```java
try { 
	SSLContext sslContext = SSLContext.getInstance("SSL");

	// set up a TrustManager that trusts everything
	sslContext.init(TrustManager[] trustAllTrustManager = new TrustManager[] {
		new X509TrustManager() {
			public java.security.cert.X509Certificate[] getAcceptedIssuers() {
				return null;
			}
			
			@Override
			public void checkClientTrusted(X509Certificate[] chain, String authType) {}

			@Override
			public void checkServerTrusted(X509Certificate[] chain, String authType){}
		}
	});
} catch(Exception exception) {
	exception.printStackTrace();
}
```

The following code is used to disable hostname validation of a certificate:
```java
try {
	SSLContext sslContext = SSLContext.getInstance("SSL");
	
	HostnameVerifier allHostnameVerifier = new HostnameVerifier() {
		public boolean verify(String hostname, SSLSession session) {
			return true;
		}
	};
	
	HttpsURLConnection.setDefaultSSLSocketFactory(sslContext.getSocketFactory());
	HttpsURLConnection.setDefaultHostnameVerifier(allHostnameVerifier);
} catch(Exception exception) {
	exception.printStackTrace();
}
```

### Apache HttpClient
Search for:
* TrustAllStrategey
* TrustSelfSignedStrategy
* NoopHostnameVerifier
* AllowAllHostnameVerifier

The following code will disable CA validation (i.e. trust any signed certificate):
```java
sslContextBuilder.loadTrustMaterial(new TrustAllStrategy());
```

The following code will trust all self-signed certificates:
```java
sslContextBuilder.loadTrustMaterial(new TrustSelfSignedStrategy());
```

The code will look something similar to(https://stackoverflow.com/questions/2703161/how-to-ignore-ssl-certificate-errors-in-apache-httpclient-4-0):
```java
CloseableHttpClient httpClient = HttpClients
                                .custom()
                                .setHostnameVerifier(AllowAllHostnameVerifier.INSTANCE)
                                .build();
```
Or

```java
CloseableHttpClient httpClient = HttpClients
                                .custom()
                                .setSSLHostnameVerifier(NoopHostnameVerifier.INSTANCE)
                                .build();
```

## Python
### Requests Library

Certificate validation is disabled by using the verify argument. For example (https://stackoverflow.com/questions/10667960/python-requests-throwing-sslerror):
```python
requests.get('https://example.com', verify=False)
```