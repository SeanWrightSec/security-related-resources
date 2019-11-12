# Certificate Validation Disabled Source Code Checks
This file holds common code which is used to disable certificate validation. By searching for this code in your source code you will hopefully be able to identify code in your systems which has had certificate validation disabled.

## Java
### Standard API

The following code is used to disable CA validation of a certificate:
```java
try { 
	SSLContext sslContext = SSLContext.getInstance("SSL");
	
	// set up a TrustManager that trusts certificates signed by any CA
	TrustManager[] trustAllTrustManager = {
		new X509TrustManager() {
			public X509Certificate[] getAcceptedIssuers() {
				return null;
			}
			
			public void checkClientTrusted(X509Certificate[] chain, String authType) { }

			public void checkServerTrusted(X509Certificate[] chain, String authType) { }
		}
	};
	
	sslContext.init(null, trustAllTrustManager, new SecureRandom());
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
Search for:
* TrustAllStrategey
* TrustSelfSignedStrategy
* NoopHostnameVerifier
* AllowAllHostnameVerifier

The following code will disable CA validation (i.e. trust any signed certificate):
```java
SSLContext sslContext = SSLContexts.custom()
				   .loadTrustMaterial(new TrustAllStrategy())
				   .build();
SSLConnectionSocketFactory sslContextFactory = SSLConnectionSocketFactor(sslContext, SSLConnectionSocketFactory.getDefaultHostnameVerifier());
```

The following code will trust all self-signed certificates:
```java
SSLContext sslContext = SSLContexts.custom()
				   .loadTrustMaterial(new TrustSelfSignedStrategy())
				   .build();
SSLConnectionSocketFactory sslContextFactory = SSLConnectionSocketFactor(sslContext, SSLConnectionSocketFactory.getDefaultHostnameVerifier());
```

To ignore hostname validation, the code will look something similar to(https://stackoverflow.com/questions/2703161/how-to-ignore-ssl-certificate-errors-in-apache-httpclient-4-0):
```java
CloseableHttpClient httpClient = HttpClients.custom()
                                	    .setSSLHostnameVerifier(AllowAllHostnameVerifier.INSTANCE)
                                	    .build();
```
Or

```java
CloseableHttpClient httpClient = HttpClients.custom()
                                	    .setSSLHostnameVerifier(NoopHostnameVerifier.INSTANCE)
                                	    .build();
```

## Python
### Requests Library

Certificate validation is disabled by using the verify argument. For example (https://stackoverflow.com/questions/10667960/python-requests-throwing-sslerror):
```python
requests.get('https://example.com', verify=False)
```
