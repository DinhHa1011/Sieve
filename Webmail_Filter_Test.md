# Sieve Rouncube

## Duplicate ERROR 
- Edit filter set báo lỗi
```
require ["duplicate","fileinto","mailbox"];
if allof (duplicate :header "Subject")
{fileinto :create "INBOX.AHa";}
```
![](https://hackmd.io/_uploads/Byprw-PUh.png) 
## Editheader ERROR  
- Edit filter set báo lỗi
![](https://hackmd.io/_uploads/H1jH9ZPIh.png)

## Notification OK
Add filter trong roundcube. 
![](https://hackmd.io/_uploads/B1-vBZw8n.png)


## Include OK 
- Edit filter set 
![](https://hackmd.io/_uploads/rkat3Mv82.png)
- Filter được sinh ra từ đoạn script trên
![](https://hackmd.io/_uploads/SkRihMPUn.png)
- Kết quả flag.sieve (subject là `hi` thì Flagged) được thực thi
![](https://hackmd.io/_uploads/ByMOpzvIn.png)


## Spam and virus test ERROR
- Edit filter set báo lỗi
![](https://hackmd.io/_uploads/HybWnGw83.png)


## Vacation OK 
Add filter trong roundcube. 
![](https://hackmd.io/_uploads/HJgKYxvLn.png)
Log
![](https://hackmd.io/_uploads/Bkf1clvLn.png)


## Variables OK
- Edit filter set 
![](https://hackmd.io/_uploads/By1v8WwLh.png)

![](https://hackmd.io/_uploads/HJOxxZvIn.png)
- Filter được sinh ra từ đoạn script trên
![](https://hackmd.io/_uploads/Bk3ZeZw82.png)

- Log
![](https://hackmd.io/_uploads/B1zXg-PL2.png)

## Extdata ERROR
- Edit filter set báo lỗi
![](https://hackmd.io/_uploads/HJODsZD83.png)

## Extprogram OK
- Edit filter set
![](https://hackmd.io/_uploads/BJlLk8MvL2.png)
- Log
![](https://hackmd.io/_uploads/HJb8LzP8h.png)

## Imapsieve
- Sieve script cấu hình bên server ko làm được trên webmail
