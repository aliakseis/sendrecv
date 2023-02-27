# sendrecv
The pkg-config executable from the gstreamer installation must be in the path.
```
python simple_server.py --disable-ssl
sendrecv.exe  --our-id=123 --server=ws://127.0.0.1:8443  --disable-ssl
sendrecv.exe  --peer-id=123 --server=ws://127.0.0.1:8443  --disable-ssl
```
or using https://ntfy.sh/
```
sendrecv.exe  --session-id=123 --autovideosrc --autoaudiosrc
sendrecv.exe  --session-id=123
```
