# Cookie Encryption Gateway

Encrypt any cookies leaving this virtual server and decrypt them on the way in.  This can also be done within the HTTP Profile, but with this all cookies are encrypted, rather than manually specifying each cookie to encrypt.

# F5 Cookie Encryption Gateway
```
when RULE_INIT {
  # Exposed passphrase, but this key can be synchronized to the peer LTM
    set ::passphrase "secret"

  # Private passphrase, but it isn't synchronized.  On LTM failover to
  # its peer, applications relying on the encrypted cookies will break.
    # set ::passphrase [AES::key]
}

when HTTP_REQUEST {
  foreach { cookieName } [HTTP::cookie names] {
    HTTP::cookie decrypt $cookieName ::passphrase
  }
}

when HTTP_RESPONSE {
  foreach { cookieName } [HTTP::cookie names] {
    HTTP::cookie encrypt $cookieName ::passphrase
  }
}

# Avi Cookie Encryption Gateway


```
content = avi.http.get_path()
file_types = {".gif", ".jpg", ".png", ".ico", ".css"}

for typeCount = 1, #file_types do
  if string.endswith(content, file_types[typeCount]) then
    avi.pool.select("Cache_Servers")
  else
    avi.pool.select("App_Servers")
  end
end
```


Apply this to the "HTTP Request" Event

```
cookies, count = avi.http.get_cookie_names()
if count > 0 then
  for cookie_num = 1, #cookies do
    key = 234LJKH43J3H4K6KJH77H234
    cookie_name = cookies[cookie_num]
    cookie_data = avi.http.get_cookie(cookie_name)
    decrypt = avi.crypto.decrypt(cookie_data, key)
    avi.http.replace_cookie(cookie_name, decrypt)
  end
end
```

Apply this to the "HTTP Response" Event

```
cookies, count = avi.http.get_cookie_names()
if count > 0 then
  for cookie_num = 1, #cookies do
    key = 234LJKH43J3H4K6KJH77H234
    cookie_name = cookies[cookie_num]
    cookie_data = avi.http.get_cookie(cookie_name)
    encrypt = avi.crypto.encrypt(cookie_data, key)
    avi.http.replace_cookie(cookie_name, encrypt)
  end
end
```