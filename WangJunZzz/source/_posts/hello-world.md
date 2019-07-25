{
  "service": {
    "id": "oauth",
    "name": "dev-oauth",
    "tags": ["fanyou"],
    "address": "http://dev.oauth.fanyouvip.com",
    "port": 80,

    "checks": [
      {
        "HTTP":"http://dev.oauth.fanyouvip.com",
        "interval": "10s"
      }
    ]
  }
}