# R2 Storage: Client Ownership & Developer Access

## TL;DR
**YES - Client should own their R2 account and pay their own fees.**  
**YES - You as developer get full access via API keys or team member access.**

This is the standard, professional approach for web development.

---

## Option 1: Client Owns R2 Account (RECOMMENDED)

### Setup Flow
```
1. Client creates Cloudflare account (free)
2. Client creates R2 bucket
3. Client generates API tokens
4. Client shares credentials with you (developer)
5. You configure WordPress plugin with their credentials
6. Client pays Cloudflare directly each month
```

### Access Methods for Developer

#### Method A: API Tokens (Most Common)
```
Client Dashboard → R2 → Manage R2 API Tokens → Create Token

Token Permissions:
✓ Object Read & Write
✓ Bucket Read
✓ Specific bucket only (security best practice)

Client shares with you:
- Account ID
- Access Key ID  
- Secret Access Key
- Bucket Name

You add to wp-config.php:
define('WRAITH_R2_ACCOUNT_ID', 'abc123...');
define('WRAITH_R2_ACCESS_KEY', 'xyz789...');
define('WRAITH_R2_SECRET_KEY', '...');
```

#### Method B: Team Member Access (Better for Agencies)
```
Client Dashboard → Members → Invite Member → your@email.com

Access Level:
- Administrator (full access)
- OR R2 Edit (scoped to R2 only)

You login with your own Cloudflare account
You access their R2 from your dashboard
Generate API tokens yourself
```

### Pros
✅ **Client owns their data** (important for long-term)  
✅ **Client pays directly** (no billing friction)  
✅ **Clean separation** (each client separate)  
✅ **Professional** (industry standard)  
✅ **You still have full access** via keys  
✅ **Scalable** (works for 1 or 100 clients)  
✅ **No liability** if client stops paying you  
✅ **Easy handoff** if they change developers  

### Cons
❌ Client needs to set up account (5 min extra work)  
❌ Slight coordination needed for initial setup  

### Cost Transparency
Client sees their own Cloudflare bill:
```
R2 Storage: $15.00 (1TB)
Operations: $0.50
Total: $15.50/month
```

---

## Option 2: You Own R2 Account (Not Recommended)

### Setup Flow
```
1. You create R2 bucket on your Cloudflare account
2. You configure WordPress with your credentials
3. You pay Cloudflare
4. You bill client monthly (markup or pass-through)
```

### Pros
✅ Simpler initial setup  
✅ You control everything  
✅ Can markup storage costs  

### Cons
❌ **You're liable for payment** (even if client doesn't pay)  
❌ **All client data in YOUR account** (messy)  
❌ **Billing friction** (you invoice, chase payments)  
❌ **Hard to separate** if relationship ends  
❌ **Doesn't scale** (10 clients = chaos)  
❌ **Data ownership unclear** (whose data is it?)  
❌ **Can't transfer** easily if they leave  

### When This Makes Sense
- Tiny project (personal blog, hobby site)
- You're hosting everything already
- Client explicitly requests it
- Short-term/prototype project

---

## Option 3: Hybrid (Best for Agencies)

### Setup Flow
```
Each client owns their R2 account
You're added as team member to all client accounts
You can access any client's R2 from your dashboard
Clean separation, full access
```

### Agency Dashboard View
```
Your Cloudflare Account:
├── Your personal projects
└── Team Access:
    ├── Client A (R2 Edit access)
    ├── Client B (R2 Edit access)
    └── Client C (R2 Edit access)
```

### Pros
✅ All benefits of Option 1  
✅ **Single login** to access all clients  
✅ **Professional** agency setup  
✅ Easy to onboard new clients  
✅ Easy to offboard if they leave  

---

## Developer Access Levels

### What You Can Do With API Tokens

**Full R2 Access Token:**
- Upload files to bucket
- Delete files from bucket
- List bucket contents
- Read object metadata
- Configure bucket settings

**Read-Only Token (for frontend CDN):**
- Only read/download files
- Cannot upload or delete
- Good for public assets

### What You Can Do As Team Member

**Administrator Role:**
- Everything (including billing)
- Create/delete buckets
- Manage other team members
- View usage/costs

**R2 Edit Role (Recommended):**
- Full R2 access (upload/delete)
- Cannot see billing
- Cannot manage team members
- Scoped to just R2

---

## Security Best Practices

### For Client-Owned Accounts

```php
// wp-config.php (not in Git!)
define('WRAITH_R2_ACCOUNT_ID', 'client-account-id');
define('WRAITH_R2_ACCESS_KEY', 'limited-scope-key');
define('WRAITH_R2_SECRET_KEY', 'secret-here');
define('WRAITH_R2_BUCKET', 'client-videos');
```

**Key Management:**
1. Use **bucket-specific tokens** (not account-wide)
2. **Rotate keys** every 90 days
3. **Never commit** to Git (use wp-config.php)
4. **Revoke access** when project ends
5. Use **separate tokens** for dev/staging/production

### Token Scoping Example
```
Token Name: "WordPress Production Upload"
Permissions: 
  ✓ Object Read & Write
  ✓ Bucket: wraith-videos (specific bucket only)
  ✓ Duration: 1 year
  
Token Name: "WordPress CDN Read"  
Permissions:
  ✓ Object Read only
  ✓ Bucket: wraith-videos
  ✓ Duration: Indefinite
```

---

## Real-World Workflow

### Initial Setup (One Time)

**Step 1: Client Creates Account (5 min)**
```
1. Client goes to cloudflare.com
2. Signs up (free)
3. Verifies email
4. Navigates to R2
5. Creates bucket "mysite-videos"
```

**Step 2: Client Generates API Token (2 min)**
```
1. R2 → Manage R2 API Tokens
2. Create API Token
3. Select: Object Read & Write
4. Bucket: mysite-videos (specific)
5. Copy token details
```

**Step 3: Client Shares Credentials (1 min)**
```
Email or 1Password share:
- Account ID: abc123...
- Access Key: xyz789...
- Secret Key: ...
- Bucket Name: mysite-videos
```

**Step 4: You Configure WordPress (2 min)**
```php
// Add to wp-config.php
define('WRAITH_R2_ACCOUNT_ID', 'abc123...');
define('WRAITH_R2_ACCESS_KEY', 'xyz789...');
define('WRAITH_R2_SECRET_KEY', '...');
define('WRAITH_R2_BUCKET', 'mysite-videos');
```

**Total Time: 10 minutes**

### Ongoing Management

**You (Developer):**
- Upload videos via WordPress admin
- Delete videos via WordPress admin
- Monitor storage usage
- Troubleshoot upload issues
- Update plugin features

**Client:**
- Gets monthly bill from Cloudflare
- Sees usage in their dashboard
- Owns all their video data
- Can revoke your access anytime
- Can migrate to new developer easily

---

## Cost Structure

### Client Pays Cloudflare Directly
```
Month 1:
- Storage: 50GB = $0.75
- Operations: ~$0.10
- Total: $0.85

Month 6:
- Storage: 500GB = $7.50
- Operations: ~$0.50
- Total: $8.00

Month 12:
- Storage: 1TB = $15.00
- Operations: ~$1.00
- Total: $16.00
```

**Client receives invoice directly from Cloudflare**  
**Paid via credit card on file**  
**No involvement from you**

### You (Developer) Bill For
- Plugin development (one-time)
- Monthly maintenance/support (optional)
- Custom features (hourly or project)
- Storage costs NOT included

---

## Common Client Questions

### "Do I need a Cloudflare account?"
**Yes** - It's free to create and only takes 5 minutes. You'll only pay for what you use (starts at <$1/month).

### "Why can't you just handle the storage?"
Professional web development separates infrastructure (client-owned) from services (developer-provided). You own your data and domain, just like you own your storage.

### "How much will this cost me?"
Starting around $1-5/month depending on video library size. No egress fees unlike AWS. See your exact usage in your Cloudflare dashboard anytime.

### "Can I switch developers later?"
Yes! You own the Cloudflare account and data. Just revoke the current developer's API tokens and generate new ones for a new developer.

### "What if I stop paying Cloudflare?"
Videos will stop working on your site (just like if you stopped paying for web hosting). Your data isn't deleted immediately - you'll have time to export or pay the bill.

### "Do you have access to my billing info?"
No. With API tokens, I only have access to upload/manage videos. With team member access, I can choose "R2 Edit" role which excludes billing visibility.

---

## Alternative: Cloudflare Stream (Managed Service)

Instead of R2 + custom plugin, use Cloudflare Stream:

```
Cloudflare Stream (All-in-One):
- Client creates Stream account
- Upload via dashboard or API  
- Automatic transcoding to HLS
- Built-in player
- Analytics included
- $1/1000 minutes stored
- $1/1000 minutes delivered

Cost example:
- 100 videos (10min avg) = $1/month storage
- 1000 views/month = $10/month delivery
- Total: ~$11/month
```

**Pros:**
- No plugin development needed
- Adaptive streaming built-in
- Analytics dashboard
- Easier for client

**Cons:**
- More expensive than raw R2
- Less customization
- Vendor lock-in

**When to use:** Client wants turnkey solution and has budget

---

## Recommendation Matrix

| Client Type | Recommendation | Why |
|------------|----------------|-----|
| **Professional Business** | Client owns R2, you get team access | Professional, scalable, clean |
| **Small Local Business** | Client owns R2, they share API keys | Simple, low-touch |
| **Personal/Hobby Site** | Your R2 account (maybe) | Simplicity, they may not want to manage |
| **Agency Portfolio** | Each client owns, you're team member | Scalable, professional |
| **High-Budget Client** | Cloudflare Stream | Turnkey, they want managed |

---

## Step-by-Step Client Onboarding

### Email Template for Client

```
Subject: Video Platform Setup - Cloudflare R2 Account Needed

Hi [Client],

For your video hosting platform, we'll be using Cloudflare R2 storage. 
This is the most cost-effective solution ($15/TB vs $113/TB on AWS).

You'll need to create a free Cloudflare account and R2 bucket. 
This ensures you own your data and pay Cloudflare directly.

Setup takes ~10 minutes. Here's what to do:

1. Create Cloudflare account: https://dash.cloudflare.com/sign-up
2. Navigate to R2 Object Storage
3. Create bucket named: "yoursite-videos"
4. Generate API token (I'll help with this step)
5. Share credentials with me securely

Expected cost: $1-5/month to start, scales with usage.

I'll need:
- Account ID
- Access Key ID
- Secret Access Key
- Bucket Name

I can walk you through this on a quick call if helpful.

Thanks,
[Your Name]
```

### Zoom Call Checklist (15 min)
- [ ] Client creates Cloudflare account
- [ ] Verify email together
- [ ] Navigate to R2
- [ ] Create bucket together
- [ ] Generate API token together
- [ ] Test token in WordPress
- [ ] Upload test video
- [ ] Show client their dashboard

---

## Summary

### ✅ RECOMMENDED APPROACH

**Client Owns R2 Account:**
- Professional and scalable
- Client pays Cloudflare directly
- You get full access via API tokens or team membership
- Clean data ownership
- Easy to transfer if they switch developers
- Industry standard practice

**Your Access:**
- API tokens for WordPress plugin
- OR team member access for dashboard
- Full read/write permissions
- Can manage buckets and files
- Cannot (optionally) see billing

**This is how professionals do it.**

### 🔑 Key Takeaway

You're building the house (plugin), they're buying the land (storage). 

Clean separation = professional relationship.

---

## Next Steps

1. **Decide:** Client-owned R2 (recommended) or your account
2. **If client-owned:** Send onboarding email template
3. **Schedule:** 15-min call to set up together
4. **Test:** Upload test video to verify access
5. **Document:** Save credentials in password manager
6. **Build:** Start plugin development

**Questions?** Review the client FAQ section above.

---

## Developer Resources

- [Cloudflare R2 Docs](https://developers.cloudflare.com/r2/)
- [R2 API Documentation](https://developers.cloudflare.com/r2/api/s3/)
- [Team Member Permissions](https://developers.cloudflare.com/fundamentals/account-and-billing/members/)
- [API Token Best Practices](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/)

---

**Built by RT7 Media | Wraith Video Platform**
