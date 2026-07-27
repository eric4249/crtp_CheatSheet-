>>> SID後綴快速記憶重點: 
  500 = 內建 Administrator(帳號)
  512 = Domain Admins(群組)
  519 = Enterprise Admins(Forest)
  502 = krbtgt
  544 = 本地 Administrators
  普通 user 的 RID 永遠是動態的，不會因為變 DA 而變成 500
  OU / GPO 幾乎都看 GUID 或 DN，很少用固定 SID
  實戰優先記：500, 512, 519, 502, 544, 548(Account Operators)


>>> ACL包含ACE(以只有一個ACE舉例): 
Object（例如 Domain、OU、User、Group...）
  └── DACL (ACL)
        └── ACE
              ├── Trustee SID        ← 這裡放 ...-512（代表 Domain Admins 群組）
              ├── Access Rights      ← GenericAll / WriteDacl / DCSync 等
              ├── Ace Type           ← Allow 或 Deny
              └── 其他旗標、ObjectType GUID...

>>> ACL實際上會包含多個ACE: 
>>> 一個ACE Trustee SID的部分中只有一個SID, 
>>> 用以標識誰可以用這個ACL, 或者說ACE來做什麼
資源物件 (有自己的 objectSID)
  └── DACL / SACL   ← 這是 ACL（沒有 SID）
        ├── ACE 1
        │     ├── Trustee SID     ← 只有「一個誰」（例如 ...-512）
        │     ├── Rights          ← GenericAll / WriteDacl / DCSync...
        │     ├── AceType         ← Allow / Deny
        │     └── 其他旗標、ObjectType GUID...
        ├── ACE 2
        │     ├── Trustee SID     ← 另一個誰（例如 ...-500, 即admin 或 ...-）
        │     └── ...
        └── ACE 3 ...

>>> 通用格式說明: 
  >>>> # Domain-relative SID 結構: 
    S-1-5-21-xxxx-xxxx-xxxx-RID
    | S-1-5-21 | - | Domain SID (3 parts) | - | RID |

  >>>> # 不重要/變動部分用 x 省略
    # Domain SID 每個網域不同，RID 對 Well-Known 物件固定


>>> 1. 內建高權限帳號 / 群組（Domain-relative）

  >>>> Domain 內建 Administrator 帳號: 
    S-1-5-21-xxxx-xxxx-xxxx-500
    | S-1-5-21 |-| DomainSID |-| 500 |
    特徵：Domain 內建 Administrator 帳號（User 物件）。永遠只有一個，Well-Known RID。Golden Ticket / DCSync 首要目標。

  >>>> Guest 帳號: 
    S-1-5-21-xxxx-xxxx-xxxx-501
    | S-1-5-21 |-| DomainSID |-| 501 |
    特徵：Guest 帳號。通常停用。

  >>>> krbtgt 帳號: 
    S-1-5-21-xxxx-xxxx-xxxx-502
    | S-1-5-21 |-| DomainSID |-| 502 |
    特徵：krbtgt 帳號。Golden Ticket 必要（需要其 hash）。永遠存在且 critically system。

  >>>> Domain Admins 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-512
    | S-1-5-21 |-| DomainSID |-| 512 |
    特徵：Domain Admins 群組（Group 物件）。最核心的 DA 群組。成員自動擁有高權限。

  >>>> Domain Users 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-513
    | S-1-5-21 |-| DomainSID |-| 513 |
    特徵：Domain Users 群組。所有 domain user 預設屬於此群組。

  >>>> Domain Guests 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-514
    | S-1-5-21 |-| DomainSID |-| 514 |
    特徵：Domain Guests 群組。

  >>>> Domain Computers 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-515
    | S-1-5-21 |-| DomainSID |-| 515 |
    特徵：Domain Computers 群組。

  >>>> Domain Controllers 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-516
    | S-1-5-21 |-| DomainSID |-| 516 |
    特徵：Domain Controllers 群組。所有 DC 電腦帳號屬於此。

  >>>> Schema Admins 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-518
    | S-1-5-21 |-| DomainSID |-| 518 |
    特徵：Schema Admins 群組。可修改 Schema（極高權限，Forest 等級）。

  >>>> Enterprise Admins 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-519
    | S-1-5-21 |-| DomainSID |-| 519 |
    特徵：Enterprise Admins 群組。Forest 最高權限群組之一（跨 Domain）。

  >>>> Group Policy Creator Owners 群組: 
    S-1-5-21-xxxx-xxxx-xxxx-520
    | S-1-5-21 |-| DomainSID |-| 520 |
    特徵：Group Policy Creator Owners 群組。可建立 GPO。


>>> 2. BUILTIN 群組（機器本地，非 Domain SID）

  >>>> Administrators（本地）: 
    S-1-5-32-544
    | S-1-5-32 |-| 544 |
    特徵：Administrators（本地 Administrators 群組）。Domain Admins 預設是其成員。

  >>>> Users（本地）: 
    S-1-5-32-545
    | S-1-5-32 |-| 545 |
    特徵：Users（本地 Users 群組）。

  >>>> Guests（本地）: 
    S-1-5-32-546
    | S-1-5-32 |-| 546 |
    特徵：Guests。

  >>>> Power Users: 
    S-1-5-32-547
    | S-1-5-32 |-| 547 |
    特徵：Power Users（較少用）。

  >>>> Account Operators: 
    S-1-5-32-548
    | S-1-5-32 |-| 548 |
    特徵：Account Operators。可管理帳號（Forest 常見提權路徑）。

  >>>> Server Operators: 
    S-1-5-32-549
    | S-1-5-32 |-| 549 |
    特徵：Server Operators。

  >>>> Print Operators: 
    S-1-5-32-550
    | S-1-5-32 |-| 550 |
    特徵：Print Operators。

  >>>> Backup Operators: 
    S-1-5-32-551
    | S-1-5-32 |-| 551 |
    特徵：Backup Operators。可備份/還原（含敏感檔案）。


>>> 3. 其他重要 Well-Known SIDs（通用）

  >>>> Everyone: 
    S-1-1-0
    | S-1-1-0 |
    特徵：Everyone。幾乎所有人都屬於。

  >>>> Authenticated Users: 
    S-1-5-11
    | S-1-5-11 |
    特徵：Authenticated Users。已驗證使用者。

  >>>> Enterprise Domain Controllers: 
    S-1-5-9
    | S-1-5-9 |
    特徵：Enterprise Domain Controllers。

  >>>> Local System（SYSTEM）: 
    S-1-5-18
    | S-1-5-18 |
    特徵：Local System（SYSTEM）。

  >>>> Local Service: 
    S-1-5-19
    | S-1-5-19 |
    特徵：Local Service。

  >>>> Network Service: 
    S-1-5-20
    | S-1-5-20 |
    特徵：Network Service。

  >>>> Enterprise Read-only Domain Controllers: 
    S-1-5-21-xxxx-xxxx-xxxx-498
    | S-1-5-21 |-| DomainSID |-| 498 |
    特徵：Enterprise Read-only Domain Controllers。

  >>>> Read-only Domain Controllers: 
    S-1-5-21-xxxx-xxxx-xxxx-521
    | S-1-5-21 |-| DomainSID |-| 521 |
    特徵：Read-only Domain Controllers。


>>> 4. OU / GPO / ACL 相關說明（無固定 RID）

  >>>> OU（Organizational Unit）: 
    無固定 Well-Known RID。
    特徵：每個 OU 有自己的 objectSID（動態）+ objectGUID。主要用 distinguishedName（DN）或 objectGUID 識別。ACL 常設在 OU 上控制繼承。

  >>>> GPO（Group Policy Object）: 
    無固定 RID。
    特徵：以 objectGUID 為主要識別（例如 {31B2F340-016D-11D2-945F-00C04FB984F9} 是 Default Domain Policy）。GPT 存在 SYSVOL，GPC 存在 AD。權限常透過 “Group Policy Creator Owners” 或直接 ACL 控制。

  >>>> ACL / ACE: 
    ACL 本身不是物件 SID，而是 DACL/SACL 的集合。
    特徵：
      - 每個 ACE 包含：
        - Trustee SID（誰被授權，例如 DomainSID-512）
        - AccessMask / Rights（GenericAll, WriteDacl, DCSync 等）
        - ObjectType / InheritedObjectType（GUID，針對特定屬性或物件類別）
      - 提權常找 WriteDacl / GenericAll / WriteOwner on Domain / OU / Group / User。
      - 常用：AdminSDHolder（保護 adminCount=1 物件的 ACL 模板）。
      - ACE 本身沒有 SID/RID，只記載一個 Trustee SID；ACL 是多個 ACE 的集合。


>>> 5. 其他實用動態 / 常見物件

  >>>> 普通 Domain User / 服務帳號（例如 svc-alfresco）: 
    S-1-5-21-xxxx-xxxx-xxxx-1xxx（或更高）
    | S-1-5-21 |-| DomainSID |-| 動態RID（通常 ≥1000） |
    特徵：RID 由 RID Master 動態分配，建立後永不改變。加入任何群組都不會改 SID。

  >>>> 電腦帳號（Domain Computer）: 
    S-1-5-21-xxxx-xxxx-xxxx-1xxx$
    | S-1-5-21 |-| DomainSID |-| 動態RID$ |
    特徵：結尾有 $，RID 動態。DC 電腦帳號通常是 ...-1000 左右。

  >>>> AdminSDHolder: 
    特徵：特殊容器，定期把高權限群組成員的 ACL 重設為受保護狀態（adminCount=1）。

