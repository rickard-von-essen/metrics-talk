Demo Notes
==========

Preparations
------------

```
cd sandbox
./sandbox up
```

Creating Load
=============

docker run dkuffner/docker-stress --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 90s
open http://localhost:3000

(admin/admin)

Simulate Webapp
===============

```
cd sandbox/backend
pip install --user locust
pip3 install --user -r requirements.txt

python3 main.py


# in another window

locust --host=http://localhost:8086
```

Watch the _App_ dashboard.
