private static void disableSslVerification() {
    try {
        SSLContext sc = SSLContext.getInstance("TLS");
        sc.init(null, new TrustManager[]{new X509TrustManager() {
            public void checkClientTrusted(X509Certificate[] certs, String authType) {}
            public void checkServerTrusted(X509Certificate[] certs, String authType) {}
            public X509Certificate[] getAcceptedIssuers() { return null; }
        }}, new java.security.SecureRandom());

        HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
        HttpsURLConnection.setDefaultHostnameVerifier((hostname, session) -> true);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
