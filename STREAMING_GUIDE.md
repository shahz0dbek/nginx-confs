# Streaming config line-by-line guide

This guide explains every line in `conf.d/streaming.conf`.

## Annotated configuration

```
 1  upstream streaming_origin {
 2    server 127.0.0.1:8080;
 3    keepalive 32;
 4  }
 5  
 6  server {
 7    listen 80;
 8    server_name streaming.local;
 9  
10    # Media segments and manifests served from the origin.
11    location /hls/ {
12      proxy_pass http://streaming_origin;
13      proxy_http_version 1.1;
14      proxy_set_header Host $host;
15      proxy_set_header X-Real-IP $remote_addr;
16      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
17      proxy_set_header X-Forwarded-Proto $scheme;
18  
19      # Allow players to read media from browsers.
20      add_header Access-Control-Allow-Origin "*" always;
21      add_header Access-Control-Allow-Methods "GET, HEAD, OPTIONS" always;
22      add_header Access-Control-Allow-Headers "Range" always;
23  
24      # HLS segments benefit from range requests.
25      proxy_set_header Range $http_range;
26      proxy_set_header If-Range $http_if_range;
27    }
28  
29    location /dash/ {
30      proxy_pass http://streaming_origin;
31      proxy_http_version 1.1;
32      proxy_set_header Host $host;
33      proxy_set_header X-Real-IP $remote_addr;
34      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
35      proxy_set_header X-Forwarded-Proto $scheme;
36  
37      add_header Access-Control-Allow-Origin "*" always;
38      add_header Access-Control-Allow-Methods "GET, HEAD, OPTIONS" always;
39      add_header Access-Control-Allow-Headers "Range" always;
40  
41      proxy_set_header Range $http_range;
42      proxy_set_header If-Range $http_if_range;
43    }
44  
45    location /api/ {
46      proxy_pass http://streaming_origin;
47      proxy_http_version 1.1;
48      proxy_set_header Host $host;
49      proxy_set_header X-Real-IP $remote_addr;
50      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
51      proxy_set_header X-Forwarded-Proto $scheme;
52    }
53  
54    location / {
55      root /usr/share/nginx/html;
56      index index.html;
57      try_files $uri $uri/ =404;
58    }
59  }
```

## Line-by-line explanation

1. Defines an upstream block named `streaming_origin` for reuse in proxy locations.
2. Sets the origin server to `127.0.0.1:8080` (change to your media backend host/port).
3. Keeps up to 32 idle keepalive connections to the origin for reuse.
4. Ends the upstream block.
5. Blank line for readability.
6. Starts a server block.
7. Listens for HTTP traffic on port 80.
8. Matches requests for the `streaming.local` hostname.
9. Blank line for readability.
10. Comment describing the HLS section purpose.
11. Matches any request path starting with `/hls/` (HLS playlists/segments).
12. Proxies `/hls/` requests to the `streaming_origin` upstream.
13. Forces HTTP/1.1 to support keepalive and upstream features.
14. Forwards the original `Host` header to the upstream.
15. Passes the client IP in `X-Real-IP`.
16. Appends the client IP to `X-Forwarded-For`.
17. Sends the original scheme (`http`/`https`) via `X-Forwarded-Proto`.
18. Blank line for readability.
19. Comment describing the CORS headers.
20. Allows all origins to access media (CORS).
21. Allows `GET`, `HEAD`, and `OPTIONS` methods for players and preflight.
22. Allows the `Range` header so browsers can request byte ranges.
23. Blank line for readability.
24. Comment noting byte-range support.
25. Forwards the incoming `Range` header to the origin.
26. Forwards the incoming `If-Range` header to support conditional ranges.
27. Ends the `/hls/` location.
28. Blank line for readability.
29. Matches any request path starting with `/dash/` (DASH manifests/segments).
30. Proxies `/dash/` requests to the `streaming_origin` upstream.
31. Forces HTTP/1.1 to support keepalive and upstream features.
32. Forwards the original `Host` header to the upstream.
33. Passes the client IP in `X-Real-IP`.
34. Appends the client IP to `X-Forwarded-For`.
35. Sends the original scheme (`http`/`https`) via `X-Forwarded-Proto`.
36. Blank line for readability.
37. Allows all origins to access media (CORS).
38. Allows `GET`, `HEAD`, and `OPTIONS` methods for players and preflight.
39. Allows the `Range` header so browsers can request byte ranges.
40. Blank line for readability.
41. Forwards the incoming `Range` header to the origin.
42. Forwards the incoming `If-Range` header to support conditional ranges.
43. Ends the `/dash/` location.
44. Blank line for readability.
45. Matches API requests that should go to the origin service.
46. Proxies `/api/` requests to the `streaming_origin` upstream.
47. Forces HTTP/1.1 to support keepalive and upstream features.
48. Forwards the original `Host` header to the upstream.
49. Passes the client IP in `X-Real-IP`.
50. Appends the client IP to `X-Forwarded-For`.
51. Sends the original scheme (`http`/`https`) via `X-Forwarded-Proto`.
52. Ends the `/api/` location.
53. Blank line for readability.
54. Matches all remaining requests (static assets).
55. Sets the document root for static files.
56. Serves `index.html` by default.
57. Returns 404 if no file or directory exists.
58. Ends the `/` location.
59. Ends the server block.
