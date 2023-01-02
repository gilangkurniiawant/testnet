<p align="center">
  <img height="300" height="auto" src="https://i.ibb.co/r7g8wn6/photo-2021-05-18-09-00-47.jpg">
</p>

# Meson Network node setup for Testnet
>- Register : https://dashboard.gaganode.com/register?referral_code=bkdleeokqy

## 1️⃣ Go To Shell Console
```python
curl -o app-linux-amd64.tar.gz https://assets.coreservice.io/public/package/22/app/1.0.3/app-1_0_3.tar.gz && tar -zxf app-linux-amd64.tar.gz && rm -f app-linux-amd64.tar.gz && cd ./app-linux-amd64 && sudo ./app service install
```
## 2️⃣ Start APP
```python
./app
```
```python
start
```
### ⚠ PLEASE WAIT 30 SEC ⚠
```python
./apps/gaganode/gaganode config set --token=nczxsnqkpwnaspvwvgqerlkb
```

## 4️⃣ Restart APP
```python
./app
```
```python
restart
```

## Check Your Node Status
Go to : https://dashboard.gaganode.com/user_node <br>
Make Sure Your Node is Online <br><br>
<img height="500" height="auto" src="https://i.ibb.co/JzDGXkM/Capture.jpg" alt="1" border="0" /></a>







