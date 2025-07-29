# EduBridge - AWS S3 Static Website Hosting Guide

## Overview
This guide will help you deploy the EduBridge frontend to AWS S3 for static website hosting.

## Prerequisites
- AWS Account with appropriate permissions
- AWS CLI installed and configured
- IAM user with S3 permissions

## Files Ready for Deployment
All files in this directory are optimized for static hosting:

### Main Pages
- `index.html` - Main landing page with document upload
- `attention-detection.html` - AI attention detection feature
- `focus-racer.html` - Python learning racing game
- `game.html` - Math adventure game
- `loading.html` - Learning hub and dashboard
- `blind.html` - Accessibility features for visually impaired
- `deaf.html` - Features for hearing impaired users
- `dyslexic.html` - Dyslexia-friendly interface
- `dyslexicgame.html` - Educational games for dyslexic users

### Assets
- `main.css` - Consolidated responsive stylesheet
- `style.css` - Additional styling
- `styles.css` - Legacy styles
- `404.html` - Custom error page
- `robots.txt` - SEO robots file
- `sitemap.xml` - Site structure for search engines

## Step 1: Create S3 Bucket

1. **Create Bucket**
   ```bash
   aws s3 mb s3://your-edubridge-bucket-name --region us-east-1
   ```

2. **Enable Static Website Hosting**
   ```bash
   aws s3 website s3://your-edubridge-bucket-name \
     --index-document index.html \
     --error-document 404.html
   ```

## Step 2: Configure Bucket Policy

Create a bucket policy to allow public read access:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-edubridge-bucket-name/*"
        }
    ]
}
```

Apply the policy:
```bash
aws s3api put-bucket-policy \
  --bucket your-edubridge-bucket-name \
  --policy file://bucket-policy.json
```

## Step 3: Upload Files

Upload all files to S3:
```bash
# Upload HTML files
aws s3 cp . s3://your-edubridge-bucket-name/ \
  --recursive \
  --exclude "*.md" \
  --exclude "AWS-S3-DEPLOYMENT-GUIDE.md"

# Set correct content types
aws s3 cp main.css s3://your-edubridge-bucket-name/main.css \
  --content-type "text/css"
aws s3 cp style.css s3://your-edubridge-bucket-name/style.css \
  --content-type "text/css"
aws s3 cp styles.css s3://your-edubridge-bucket-name/styles.css \
  --content-type "text/css"
```

## Step 4: Configure MIME Types and Caching

Set appropriate headers for better performance:
```bash
# Set cache headers for CSS files
aws s3 cp s3://your-edubridge-bucket-name/main.css \
  s3://your-edubridge-bucket-name/main.css \
  --metadata-directive REPLACE \
  --cache-control "max-age=31536000" \
  --content-type "text/css"

# Set cache headers for HTML files
aws s3 cp s3://your-edubridge-bucket-name/index.html \
  s3://your-edubridge-bucket-name/index.html \
  --metadata-directive REPLACE \
  --cache-control "max-age=3600" \
  --content-type "text/html"
```

## Step 5: Optional - CloudFront Distribution

For better performance and HTTPS, create a CloudFront distribution:

1. **Create Distribution**
   ```bash
   aws cloudfront create-distribution \
     --distribution-config file://cloudfront-config.json
   ```

2. **Sample CloudFront Config** (cloudfront-config.json):
   ```json
   {
     "CallerReference": "edubridge-$(date +%s)",
     "Comment": "EduBridge Static Website",
     "DefaultRootObject": "index.html",
     "Origins": {
       "Quantity": 1,
       "Items": [
         {
           "Id": "S3-your-edubridge-bucket-name",
           "DomainName": "your-edubridge-bucket-name.s3-website-us-east-1.amazonaws.com",
           "CustomOriginConfig": {
             "HTTPPort": 80,
             "HTTPSPort": 443,
             "OriginProtocolPolicy": "http-only"
           }
         }
       ]
     },
     "DefaultCacheBehavior": {
       "TargetOriginId": "S3-your-edubridge-bucket-name",
       "ViewerProtocolPolicy": "redirect-to-https",
       "TrustedSigners": {
         "Enabled": false,
         "Quantity": 0
       },
       "ForwardedValues": {
         "QueryString": false,
         "Cookies": {
           "Forward": "none"
         }
       },
       "MinTTL": 0
     },
     "Enabled": true
   }
   ```

## Step 6: Custom Domain (Optional)

1. **Register domain in Route 53 or use existing domain**
2. **Create SSL certificate in ACM**
3. **Configure CloudFront to use custom domain**
4. **Update DNS records**

## Step 7: Update Configuration

After deployment, update the following:

1. **Update sitemap.xml** - Replace `https://your-domain.com` with actual domain
2. **Update robots.txt** - Replace sitemap URL with actual domain
3. **Update meta tags** - Replace placeholder URLs in HTML files

## Security Considerations

1. **Content Security Policy** - Add CSP headers via CloudFront
2. **HTTPS Only** - Ensure all traffic is encrypted
3. **Access Logging** - Enable S3 access logging for monitoring

## Performance Optimization

1. **Gzip Compression** - Enable in CloudFront
2. **Browser Caching** - Set appropriate cache headers
3. **Image Optimization** - Compress images before upload
4. **Minification** - Consider minifying CSS/JS files

## Monitoring and Analytics

1. **CloudWatch** - Monitor CloudFront metrics
2. **Google Analytics** - Add tracking code to HTML files
3. **AWS Cost Explorer** - Monitor costs

## Backup and Version Control

1. **S3 Versioning** - Enable bucket versioning
2. **Git Repository** - Keep source code in version control
3. **Automated Deployment** - Consider CI/CD pipeline

## Troubleshooting

### Common Issues:
1. **403 Forbidden** - Check bucket policy and permissions
2. **404 Errors** - Verify error document configuration
3. **CSS Not Loading** - Check MIME types and CORS settings
4. **Slow Loading** - Implement CloudFront caching

### Useful Commands:
```bash
# Check bucket website configuration
aws s3api get-bucket-website --bucket your-edubridge-bucket-name

# List bucket contents
aws s3 ls s3://your-edubridge-bucket-name --recursive

# Sync local changes
aws s3 sync . s3://your-edubridge-bucket-name --delete
```

## Cost Estimation

- **S3 Storage**: ~$0.023 per GB/month
- **S3 Requests**: ~$0.0004 per 1,000 requests
- **CloudFront**: ~$0.085 per GB transferred
- **Route 53**: ~$0.50 per hosted zone/month

## Support and Maintenance

1. **Regular Updates** - Keep content and security patches current
2. **Performance Monitoring** - Regular performance audits
3. **User Feedback** - Monitor user experience and accessibility
4. **Security Audits** - Regular security assessments

---

**Note**: Replace `your-edubridge-bucket-name` and `your-domain.com` with your actual bucket name and domain throughout this guide.

For additional support, refer to AWS documentation or contact the EduBridge development team.
