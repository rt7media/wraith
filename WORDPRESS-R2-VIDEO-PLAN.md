# WordPress Video System with R2 Storage - Implementation Plan

## Overview
Build a WordPress-based video hosting platform using Cloudflare R2 storage and custom post types with integrated video players from Wraith demos.

## Architecture Options

### Option A: Full Custom Plugin (RECOMMENDED)
**Best for:** Complete control, client sites, scalability

**Components:**
- Custom post type: `video`
- R2 direct upload integration
- Admin interface for video management
- Frontend player integration (Video.js or Plyr)
- Metadata management (duration, dimensions, file size)

**Pros:**
- Complete control over workflow
- No dependency on third-party plugins
- Optimized for R2
- Can add custom features easily

**Cons:**
- More development time
- Need to handle transcoding separately

---

### Option B: Extend Existing Media Library
**Best for:** Quick setup, familiar WordPress UX

**Components:**
- Hook into WordPress media library
- Override upload to send to R2 instead of local
- Custom metadata fields for video
- Player shortcode system

**Pros:**
- Familiar WordPress interface
- Less code to write
- Users already know the workflow

**Cons:**
- Limited customization
- Harder to add video-specific features
- Media library not optimized for large files

---

### Option C: Hybrid Approach (BEST BALANCE)
**Best for:** Most projects, flexibility + ease of use

**Components:**
- Custom post type for videos (catalog/organization)
- Custom uploader bypassing media library (for large files)
- Integration with media library for thumbnails
- Template system for display

**Pros:**
- Best of both worlds
- Familiar UI where it makes sense
- Custom workflow for videos
- Easy to extend

**Cons:**
- Slightly more complex architecture

---

## Recommended Tech Stack

### Core Plugin
```
wraith-video-platform/
├── includes/
│   ├── class-video-post-type.php
│   ├── class-r2-uploader.php
│   ├── class-video-metadata.php
│   ├── class-player-renderer.php
│   └── class-admin-interface.php
├── admin/
│   ├── css/
│   ├── js/
│   └── views/
├── public/
│   ├── css/
│   ├── js/
│   └── templates/
└── wraith-video-platform.php
```

### Video Player Choice
**Recommendation: Plyr**
- Lightweight
- Beautiful UI out of the box
- Easy WordPress integration
- HLS support via plugin
- Good mobile experience

**Alternative: Video.js**
- If you need HLS/DASH streaming
- More advanced features
- Plugin ecosystem

### R2 Integration
**Cloudflare R2**
- S3-compatible API
- No egress fees (huge savings)
- Fast CDN delivery
- Simple pricing: $0.015/GB storage

---

## Database Schema

### Custom Post Type: `video`
```php
'video' => [
    'labels' => [
        'name' => 'Videos',
        'singular_name' => 'Video'
    ],
    'supports' => ['title', 'editor', 'thumbnail', 'author', 'comments'],
    'public' => true,
    'has_archive' => true,
    'rewrite' => ['slug' => 'videos']
]
```

### Custom Meta Fields
```php
_video_r2_url          // R2 object URL
_video_r2_key          // R2 object key
_video_duration        // Duration in seconds
_video_file_size       // File size in bytes
_video_dimensions      // Width x Height
_video_mime_type       // video/mp4, etc.
_video_upload_date     // Upload timestamp
_video_status          // processing, ready, error
_video_views           // View counter
_video_hls_url         // HLS manifest URL (if transcoded)
```

### Custom Taxonomies
```php
- video_category    // Documentary, Tutorial, Promo, etc.
- video_tag         // Standard tags
- video_collection  // Playlists/Series
```

---

## R2 Upload Workflow

### Step 1: Admin Upload Interface
```javascript
// Drag & drop uploader
1. User uploads video file
2. JavaScript chunks file (for large uploads)
3. Progress bar shows upload status
4. Creates WordPress post in "processing" state
```

### Step 2: Backend Processing
```php
// PHP handles R2 upload
1. Receive chunked upload
2. Validate file (type, size)
3. Upload to R2 using AWS SDK
4. Generate unique key: videos/YYYY/MM/post-id-video-slug.mp4
5. Store R2 URL and metadata
6. Update post status to "ready"
```

### Step 3: Optional Transcoding
```
Options:
A) Cloudflare Stream (paid, easiest)
B) FFmpeg via cron job
C) External service (Mux, AWS MediaConvert)
D) Client-side transcode before upload
```

---

## Frontend Display

### Single Video Template
```php
// single-video.php
- Video player (Plyr/Video.js)
- Title and description
- Metadata (duration, upload date, views)
- Related videos
- Comments section
- Share buttons
```

### Archive Template
```php
// archive-video.php
- Grid of video thumbnails
- Filters (category, date, popularity)
- Search functionality
- Pagination
```

### Shortcode System
```php
[wraith_video id="123"]
[wraith_playlist category="tutorials"]
[wraith_latest count="6"]
```

### Gutenberg Block
```javascript
// Video block for editor
- Select video from library
- Player size options
- Autoplay/loop settings
- Custom thumbnail override
```

---

## R2 Configuration

### Bucket Setup
```bash
# Create R2 bucket
wrangler r2 bucket create wraith-videos

# Configure CORS
{
  "AllowedOrigins": ["https://yoursite.com"],
  "AllowedMethods": ["GET", "PUT", "POST"],
  "AllowedHeaders": ["*"],
  "MaxAgeSeconds": 3000
}
```

### WordPress Settings
```php
// wp-config.php or plugin settings
define('WRAITH_R2_ACCOUNT_ID', 'your-account-id');
define('WRAITH_R2_ACCESS_KEY', 'your-access-key');
define('WRAITH_R2_SECRET_KEY', 'your-secret-key');
define('WRAITH_R2_BUCKET', 'wraith-videos');
define('WRAITH_R2_PUBLIC_URL', 'https://your-custom-domain.com');
```

### Custom Domain (Recommended)
```
R2 Public Bucket → videos.yoursite.com
- Better branding
- No R2 URL exposure
- Easier to migrate if needed
```

---

## Key Features to Build

### Phase 1: MVP (Week 1-2)
- [ ] Custom post type registration
- [ ] Basic R2 upload (single file)
- [ ] Store video metadata
- [ ] Frontend player integration (Plyr)
- [ ] Single video template
- [ ] Archive template

### Phase 2: Enhanced UX (Week 3)
- [ ] Chunked uploads for large files
- [ ] Admin video library interface
- [ ] Thumbnail generation/upload
- [ ] Video shortcode
- [ ] Basic analytics (view count)

### Phase 3: Advanced Features (Week 4)
- [ ] HLS transcoding support
- [ ] Video collections/playlists
- [ ] Search and filtering
- [ ] Gutenberg block
- [ ] Video sitemap for SEO

### Phase 4: Premium Features (Optional)
- [ ] Video commenting system
- [ ] User video uploads (frontend)
- [ ] Membership/paywall integration
- [ ] Live streaming support
- [ ] Advanced analytics dashboard

---

## Code Examples

### Register Custom Post Type
```php
function wraith_register_video_post_type() {
    register_post_type('video', [
        'labels' => [
            'name' => 'Videos',
            'singular_name' => 'Video',
            'add_new' => 'Add New Video',
            'add_new_item' => 'Add New Video',
            'edit_item' => 'Edit Video',
        ],
        'public' => true,
        'has_archive' => true,
        'menu_icon' => 'dashicons-video-alt3',
        'supports' => ['title', 'editor', 'thumbnail', 'author'],
        'rewrite' => ['slug' => 'videos'],
        'show_in_rest' => true, // Gutenberg support
    ]);
}
add_action('init', 'wraith_register_video_post_type');
```

### R2 Upload Function
```php
use Aws\S3\S3Client;

function wraith_upload_to_r2($file_path, $file_name) {
    $s3 = new S3Client([
        'version' => 'latest',
        'region' => 'auto',
        'endpoint' => 'https://' . WRAITH_R2_ACCOUNT_ID . '.r2.cloudflarestorage.com',
        'credentials' => [
            'key' => WRAITH_R2_ACCESS_KEY,
            'secret' => WRAITH_R2_SECRET_KEY,
        ],
    ]);
    
    $key = 'videos/' . date('Y/m') . '/' . $file_name;
    
    $result = $s3->putObject([
        'Bucket' => WRAITH_R2_BUCKET,
        'Key' => $key,
        'SourceFile' => $file_path,
        'ContentType' => 'video/mp4',
    ]);
    
    return [
        'url' => $result['ObjectURL'],
        'key' => $key
    ];
}
```

### Frontend Player Output
```php
function wraith_render_video_player($post_id) {
    $r2_url = get_post_meta($post_id, '_video_r2_url', true);
    $poster = get_the_post_thumbnail_url($post_id, 'large');
    
    ?>
    <div class="wraith-video-player">
        <video id="player-<?php echo $post_id; ?>" 
               playsinline 
               controls 
               poster="<?php echo esc_url($poster); ?>">
            <source src="<?php echo esc_url($r2_url); ?>" type="video/mp4" />
        </video>
    </div>
    
    <script>
        const player = new Plyr('#player-<?php echo $post_id; ?>', {
            controls: ['play-large', 'play', 'progress', 'current-time', 
                      'mute', 'volume', 'settings', 'fullscreen'],
        });
    </script>
    <?php
}
```

---

## Cost Estimates

### Cloudflare R2
```
Storage: $0.015/GB/month
Example: 1TB = $15/month

Operations:
- Class A (write): $4.50/million
- Class B (read): $0.36/million

No egress fees! (This is huge vs S3)
```

### Comparison vs AWS S3
```
1TB storage + 1TB egress/month:
- AWS S3: ~$23 (storage) + ~$90 (egress) = $113/month
- R2: $15/month (no egress) = $15/month
Savings: $98/month (87% cheaper!)
```

---

## Security Considerations

### Upload Security
- File type validation (whitelist)
- File size limits
- Virus scanning (ClamAV integration)
- Rate limiting on uploads
- Nonce verification

### R2 Access
- Private bucket by default
- Signed URLs for protected content
- Custom domain with CloudFlare Access
- API key rotation

### WordPress Security
- Capability checks (only admins can upload)
- Sanitize all inputs
- Escape all outputs
- CSRF protection

---

## SEO Optimization

### Video Schema Markup
```php
{
  "@context": "https://schema.org",
  "@type": "VideoObject",
  "name": "Video Title",
  "description": "Video description",
  "thumbnailUrl": "https://...",
  "uploadDate": "2025-12-29",
  "duration": "PT2M30S",
  "contentUrl": "https://videos.yoursite.com/...",
}
```

### Video Sitemap
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:video="http://www.google.com/schemas/sitemap-video/1.1">
    <url>
        <loc>https://yoursite.com/videos/video-slug/</loc>
        <video:video>
            <video:title>Video Title</video:title>
            <video:thumbnail_loc>...</video:thumbnail_loc>
            <video:player_loc>...</video:player_loc>
            <video:duration>150</video:duration>
        </video:video>
    </url>
</urlset>
```

---

## Alternative Approaches

### Using Cloudflare Stream (Easier but Paid)
**Instead of raw R2:**
- Upload directly to Stream
- Automatic transcoding to HLS
- Built-in player
- Analytics included
- Cost: $1/1000 minutes stored + $1/1000 minutes delivered

**When to use:**
- Need adaptive streaming
- Don't want to handle transcoding
- Budget allows (~$5-20/month for small site)

### Using WordPress.com VIP (Enterprise)
- Built-in video hosting
- CDN included
- Automatic transcoding
- Cost: $$$$ (enterprise pricing)

### Using WP Media Library + R2 Plugin
- Use existing plugin (e.g., "Media Cloud")
- Configure for R2
- Simpler setup
- Less customization

---

## Recommended: Hybrid Custom Solution

### Why This Approach?
1. **Full control** over video workflow
2. **Cost-effective** with R2 (no egress fees)
3. **Scalable** to thousands of videos
4. **Flexible** player integration (Wraith demos)
5. **WordPress-native** admin experience

### Development Timeline
- Week 1: Post type, R2 upload, basic playback
- Week 2: Admin UI, metadata, thumbnails
- Week 3: Frontend templates, shortcodes
- Week 4: Polish, SEO, analytics

### Estimated Effort
- Solo developer: 3-4 weeks part-time
- Team: 1-2 weeks full-time
- Maintenance: 2-4 hours/month

---

## Next Steps

1. **Decide on architecture** (Custom plugin recommended)
2. **Set up R2 bucket** and get credentials
3. **Choose player** (Plyr for simplicity, Video.js for features)
4. **Build MVP** (upload → store → display)
5. **Test with real videos**
6. **Add features incrementally**

## Questions to Answer

- **Video sizes?** (Affects upload strategy)
- **Number of videos?** (Affects storage planning)
- **Transcoding needed?** (HLS/DASH for adaptive streaming)
- **Public or gated?** (Authentication requirements)
- **User uploads?** (Frontend upload form)
- **Live streaming?** (Requires different approach)

---

**Ready to start building?** Let's begin with Phase 1 MVP!
