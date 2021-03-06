# Add SameSite attribute to Cookies that do not have it

Here is iRule and DataScript which can handle multiple Set-Cookie headers in a response.  If a Set-Cookie header already has SameSite attribute present, it is passed through unmodified.

## iRule to add SameSite attribute

```
when HTTP_RESPONSE {
    # Set-Cookie header can occur multiple times, treat as list
    set num [HTTP::header count Set-Cookie]
    if {$num > 0} {
        foreach set_cookie [HTTP::header values Set-Cookie] {
            # only modify if header does not have SameSite attribute
            set foundSameSite [string match -nocase "*SameSite*" $set_cookie ]
            if {[expr {!$foundSameSite} ]} {
                set set_cookie [concat $set_cookie "; SameSite"]
            }
            # collect modified and unmodified values in list newcookies
            lappend newcookies $set_cookie
        }

        if {$num == 1} {
            # overwrite existing
            HTTP::header Set-Cookie [lindex $newcookies 0]
        } else {
            # remove and replace
            HTTP::header remove Set-Cookie
            foreach set_cookie $newcookies {
                HTTP::header insert Set-Cookie $set_cookie
            }
        }
        
    }
}
```

## DataScript to add SameSite attribute

```lua
-- HTTP_RESPONSE
headers = avi.http.get_header()
avi.http.remove_header("Set-Cookie")
for k, v in pairs(headers) do
    if (k == "set-cookie") then
        if (type(v) == "string") then
            -- If there's only one SameSite cookie header then v is a string
            -- v needs to be converted to a table
            v = {v}
        end

        for key, val in pairs(v) do
            -- only modify if Set-Cookie header does not have SameSite attribute
            new_val = val
            if not string.contains(string.lower(val), "samesite") then
                new_val = val .."; samesite=None"
            end
            if not string.contains(string.lower(new_val), "secure") then
                new_val = new_val .."; secure"
            end
            avi.http.add_header("Set-Cookie", new_val)
        end
    end
end
```
