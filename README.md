# Hospital-Network-Lab
  Home lab that simulates a hospital network. It uses Active Directory groups, a Windows file share, NTFS permissions and Group Policy to enforce role-based access to patient-related folders, alongside an Ubuntu Linux server.


---

## 1. Lab Environment

| Machine | OS | Role | Hospital Server | Windows Server 2022 | Domain controller (`mylab.local`): AD DS, DNS, DHCP, File and Storage Services |
 Hospital Team (client) | Windows 10 | Domain-joined workstation used for testing |
 HospLinuxServ1 | Ubuntu Linux | Apache2 web server and SSH, hosts `~/Hospital` files |

All machines run in **Oracle VirtualBox**, with snapshots taken along the way.

**Active Directory structure (`mylab.local`)**
- OU `Company Hosp` with sub-OUs: `Admin`, `Doctors`, `Nurses`
- Users: John Doe, Dr Jane
- Security groups: `Hospital Staff`, `G - Doctors`

---

## 2. Linux Server Verification

**Result**
- `apache2.service`: active (running), Apache HTTP Server
- `ssh.service`: active (running), OpenBSD Secure Shell server
- Working directory: `/home/hospserv/Hospital`

**Contents of `/Hospital`**
- `patient.txt`, `patient1.txt`, `patient-notes.txt`, `welcome.txt`
- `Staff` and `Patient` directories

---

## 3. Windows File Share: HospitalShare

| Setting | Value |Share path | `C:\HospitalShare` |
| Client mapping | `H:` (Hospital Share), deployed by GPO Drive Maps |
| Share permissions | Admin and Hospital Staff (the "front door") |
| NTFS permissions | Locked down per folder (the "real lock") |

### Drive mapping troubleshooting
1. The GPO Drive Map was first set to **Create**. On the client, `net use` showed no mapping.
2. I changed the action to **Replace** and enabled **Run in user's context**.
3. After a policy refresh, `net use` showed `H:` mapped to `\\WIN-5KNUCVGC5CV\HospitalShare`.
4. John Doe could then save and open `john-test.txt` on `H:`.


## 4. NTFS Permissions: Tested and Proven

**Folder: `john` / `john-test`**

| Principal | Access | Result |

| John | Full Control | Can open ✅ |
| Domain Admins | Full Control | Can open ✅ |
| Everyone / Doctors | Removed | Cannot open ❌ |

**Folder: `Dr Files`**

| Principal | Access |

| G - Doctors | Modify |
| Domain Admins | Full Control |
| Other users | No access |



## 5. Password Policy (Default Domain Policy)

| Setting | Value |

| Minimum password length | 12 characters |
| Complexity requirements | Enabled |
| Enforce password history | 24 passwords remembered |
| Maximum password age | 90 days |
| Minimum password age | 1 day |
| Reversible encryption | Disabled |



## 🧪 Testing and Proof

**Test 1: John folder**
- Logged in as John: opens immediately ✅
- Logged in as Admin: opens immediately ✅ (Domain Admins)
- Logged in as Jane (Doctor): sees the folder, access denied ✅

**Test 2: Dr Files**
- Logged in as a non-doctor user:
  `Windows cannot access \\WIN-5KNUCVGC5CV\HospitalShare\Dr Files`
  "You do not have permission to access... Contact your network administrator." ❌ (expected)

**Test 3: HospitalShare root**
- Unauthorized user opening `\\WIN-5KNUCVGC5CV\HospitalShare\`: access denied ✅

**Conclusion**
- Share permissions = front door
- NTFS permissions = real lock
- Domain Admins always keep access for management


## 🔧 Technologies Used
- Oracle VirtualBox
- Windows Server 2022: AD DS, DNS, DHCP, File and Storage Services
- Windows 10 (domain-joined client)
- Ubuntu Linux: Apache2, SSH, Terminal
- NTFS and share permissions
- Group Policy (Drive Maps, password policy)
- Active Directory users, OUs, and security groups



## 🚀 Next Steps
- Enable file access auditing
- Configure an account lockout policy
- Join the Ubuntu server to Active Directory
- Set up backups for HospitalShare


