# WordPress Video Platform - Quick Start Guide

## TL;DR
Build a custom WordPress video hosting platform using:
- **Custom Post Type**: `video` for organizing content
- **Cloudflare R2**: Cheap storage ($15/TB vs $113/TB on AWS)
- **Plyr Player**: Lightweight, beautiful video player
- **Timeline**: 3-4 weeks part-time

## Architecture (Recommended: Hybrid Approach)

```
┌─────────────────────────────────────────────────────┐
│              WordPress Admin                        │
│  ┌─────────────────────────────────────┐           │
│  │  Custom "Videos" Post Type          │           │
│  │  - Upload interface                 │           │
│  │  - Metadata fields                  │           │
│  │  - Thumbnail management             │           │
│  └─────────────────────────────────────┘           │
└───────────────────┬─────────────────────────────────┘
                    │
                    │ Upload via AWS SDK
                    ↓
          ┌─────────────────────┐
          │  Cloudflare R2      │
          │  Object Storage     │
          │  - /videos/2025/... │
          └─────────────────────┘
                    │
                    │ Public CDN URL
                    ↓
┌─────────────────────────────────────────────────────┐
│              WordPress Frontend                      │
│  ┌─────────────────────────────────────┐           │
│  │  Video Templates                    │           │
│  │  - Single video page                │           │
│  │  - Video archive/grid               │           │
│  │  - Plyr player integration          │           │
│  └─────────────────────────────────────┘           │
└─────────────────────────────────────────────────────┘
```

## 5-Minute Setup Checklist

### 1. R2 Setup (5 min)
```bash
# Create Cloudflare account (free)
# Dashboard → R2 → Create bucket: "wraith-videos"
# Generate API tokens (Account ID, Access Key, Secret)
```

### 2. WordPress Plugin Structure (5 min)
```
wp-content/plugins/wraith-video-platform/
├── wraith-video-platform.php    # Main plugin file
├── includes/
│   ├── post-type.php            # Register 'video' post type
│   ├── r2-upload.php            # Handle R2 uploads
│   └── player.php               # Plyr integration
└── admin/
    └── upload-form.php          # Admin upload interface
```

### 3. Install Dependencies (2 min)
```bash
cd wp-content/plugins/wraith-video-platform
composer require aws/aws-sdk-php
```

### 4. Configure (wp-config.php)
```php
define('WRAITH_R2_ACCOUNT_ID', 'your-account-id');
define('WRAITH_R2_ACCESS_KEY', 'your-access-key');
define('WRAITH_R2_SECRET_KEY', 'your-secret-key');
define('WRAITH_R2_BUCKET', 'wraith-videos');
```

### 5. Activate Plugin
- WordPress Admin → Plugins → Activate
- New "Videos" menu appears

## MVP Features (Week 1-2)

**Must Have:**
- [x] Register custom post type `video`
- [x] Admin upload form (drag & drop)
- [x] Upload to R2 with AWS SDK
- [x] Store R2 URL in post meta
- [x] Frontend: single-video.php template
- [x] Integrate Plyr player
- [x] Basic styling

**Code Snippets:** See WORDPRESS-R2-VIDEO-PLAN.md

## Cost Breakdown

### Cloudflare R2
- **100GB video library**: $1.50/month
- **1TB video library**: $15/month
- **No egress fees** (huge savings vs S3)

### Comparison
| Service | 1TB Storage | 1TB Egress | Total/Month |
|---------|-------------|------------|-------------|
| AWS S3 | $23 | $90 | **$113** |
| R2 | $15 | $0 | **$15** |

**Savings: 87% cheaper than AWS!**

## Player Choice

### Plyr (Recommended)
✅ Lightweight (10KB)  
✅ Beautiful UI  
✅ Easy setup  
✅ Mobile-friendly  
✅ HLS support via plugin  

### Video.js (Alternative)
✅ More features  
✅ Plugin ecosystem  
✅ Enterprise-grade  
❌ Heavier (250KB)  

## Workflow

### Admin Side
1. Navigate to **Videos → Add New**
2. Enter title and description
3. Upload video file (drag & drop)
4. Add thumbnail (featured image)
5. Select categories/tags
6. Click **Publish**

### Behind the Scenes
1. File uploads to temp directory
2. PHP uploads to R2 via AWS SDK
3. R2 returns public URL
4. URL saved to post meta `_video_r2_url`
5. File deleted from temp
6. Video ready to display

### Frontend Display
1. User visits `/videos/my-video/`
2. `single-video.php` template loads
3. Gets R2 URL from post meta
4. Renders Plyr player with video
5. Shows title, description, metadata

## Key Code Snippets

### Register Post Type (5 lines)
```php
register_post_type('video', [
    'labels' => ['name' => 'Videos', 'singular_name' => 'Video'],
    'public' => true,
    'has_archive' => true,
    'menu_icon' => 'dashicons-video-alt3',
    'supports' => ['title', 'editor', 'thumbnail']
]);
```

### Upload to R2 (15 lines)
```php
$s3 = new S3Client([
    'endpoint' => 'https://'.WRAITH_R2_ACCOUNT_ID.'.r2.cloudflarestorage.com',
    'credentials' => ['key' => WRAITH_R2_ACCESS_KEY, 'secret' => WRAITH_R2_SECRET_KEY]
]);

$result = $s3->putObject([
    'Bucket' => WRAITH_R2_BUCKET,
    'Key' => 'videos/'.date('Y/m').'/'.$filename,
    'SourceFile' => $filepath
]);

update_post_meta($post_id, '_video_r2_url', $result['ObjectURL']);
```

### Display Player (8 lines)
```php
$url = get_post_meta($post_id, '_video_r2_url', true);
?>
<video id="player" controls>
    <source src="<?php echo esc_url($url); ?>" type="video/mp4">
</video>
<script src="https://cdn.plyr.io/3.7.8/plyr.js"></script>
<script>new Plyr('#player');</script>
```

## Metadata to Store

```php
_video_r2_url       // R2 public URL
_video_r2_key       // R2 object key
_video_duration     // Duration in seconds
_video_file_size    // File size in bytes
_video_dimensions   // 1920x1080
_video_upload_date  // Upload timestamp
_video_views        // View counter
```

## Phase Roadmap

### Phase 1: MVP (Week 1-2) - Core Functionality
- Post type + R2 upload + Player

### Phase 2: Enhanced UX (Week 3)
- Chunked uploads
- Video library interface
- Shortcodes `[wraith_video id="123"]`

### Phase 3: Advanced (Week 4)
- HLS transcoding
- Collections/playlists
- Gutenberg block
- Analytics

### Phase 4: Premium (Optional)
- Frontend user uploads
- Membership paywall
- Live streaming
- Advanced analytics

## Decision Matrix

**Choose Custom Plugin if:**
- Need full control
- Multiple client sites
- Custom features planned
- Budget for development

**Choose Existing Plugin if:**
- Quick launch needed
- Standard features only
- Limited dev resources
- Prefer updates/support

**Choose Cloudflare Stream if:**
- Need adaptive streaming
- Want zero maintenance
- Budget allows ($5-20/month)
- Enterprise features

## Next Steps

1. **Set up R2 bucket** (5 minutes)
2. **Create plugin skeleton** (30 minutes)
3. **Build upload form** (2 hours)
4. **Test upload to R2** (1 hour)
5. **Create video template** (2 hours)
6. **Integrate Plyr** (1 hour)
7. **Test and refine** (2 hours)

**Total MVP Time: ~10 hours**

## Questions Before Starting

- [ ] Expected video sizes? (Affects upload strategy)
- [ ] Total videos expected? (Affects storage planning)
- [ ] Need transcoding/HLS? (Adds complexity)
- [ ] Public or membership-gated?
- [ ] Allow user uploads? (Frontend forms)
- [ ] Need live streaming? (Different approach)

## Resources

- **Full Plan**: WORDPRESS-R2-VIDEO-PLAN.md (533 lines, comprehensive)
- **Wraith Players**: /wraith repo (4 player demos)
- **R2 Docs**: https://developers.cloudflare.com/r2/
- **Plyr Docs**: https://plyr.io/

---

**Ready to build? Start with the full plan document!**
