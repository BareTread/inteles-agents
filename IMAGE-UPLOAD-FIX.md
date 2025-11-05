# 🔧 Quick Fix for inteles-image-curator Agent

## Problem Identified

The `create_media` MCP tool is failing with "Nicio dispoziție de conținut furnizată" because it requires **ALL 5 parameters** to be provided.

## Required Parameters for create_media

```python
inteles_wordpress_create_media(
    title="Image Title",
    alt_text="Alt text (50-125 characters)",
    caption="Romanian caption",
    description="Description for WordPress media library",
    source_url="file://absolute/path/to/image.webp"
)
```

## Common Mistakes That Cause the Error

### ❌ Missing Description Parameter
```python
# BROKEN - Missing description
inteles_wordpress_create_media(
    title="Title",
    alt_text="Alt text",
    caption="Caption",
    source_url="file://path/image.webp"
)
```

### ❌ Wrong Source URL Format
```python
# BROKEN - Wrong path format
inteles_wordpress_create_media(
    title="Title",
    alt_text="Alt text",
    caption="Caption",
    description="Description",
    source_url="/home/user/path/image.webp"  # Missing file://
)
```

### ❌ Missing Any Parameter
```python
# BROKEN - Missing any parameter
inteles_wordpress_create_media(
    title="Title",
    alt_text="Alt text",
    # Missing caption, description, or source_url
)
```

## ✅ Correct Implementation

```python
def upload_image_to_wordpress(image_path, title, alt_text, caption):
    """Upload image to WordPress with all required parameters"""

    # Convert to absolute path if needed
    if not image_path.startswith('file://'):
        import os
        absolute_path = os.path.abspath(image_path)
        source_url = f"file://{absolute_path}"
    else:
        source_url = image_path

    # Prepare description
    description = f"Imagine pentru articol inteles.ro: {title.lower()}"

    # Call create_media with ALL required parameters
    try:
        media_response = inteles_wordpress_create_media(
            title=title,
            alt_text=alt_text[:125],  # WordPress limit
            caption=caption,
            description=description,
            source_url=source_url
        )

        # Verify upload success
        if media_response.get("id") and media_response.get("source_url"):
            return {
                "success": True,
                "media_id": media_response["id"],
                "wordpress_url": media_response["source_url"],
                "alt_text": alt_text,
                "caption": caption
            }
        else:
            return {
                "success": False,
                "error": "Upload failed - no media ID returned"
            }

    except Exception as e:
        return {
            "success": False,
            "error": f"Upload failed: {str(e)}"
        }
```

## Debug Checklist

When `create_media` fails:

1. **Check all 5 parameters are present**:
   - ✅ title
   - ✅ alt_text
   - ✅ caption
   - ✅ description
   - ✅ source_url

2. **Verify source_url format**:
   - Must be absolute path
   - Must start with `file://`
   - File must exist at that location

3. **Check image file**:
   - File exists and is readable
   - File format supported by WordPress
   - File size reasonable (under 10MB)

4. **Test with manual upload**:
   ```python
   # Test the exact same parameters manually
   result = inteles_wordpress_create_media(
       title="Test Image",
       alt_text="Test alt text",
       caption="Test caption",
       description="Test description for WordPress",
       source_url="file:///absolute/path/to/test-image.webp"
   )
   print(result)
   ```

## Implementation Instructions for Agents

The inteles-image-curator agent should:

1. **Always include all 5 parameters** in create_media calls
2. **Use absolute paths with file:// protocol**
3. **Generate meaningful descriptions** for each image
4. **Handle errors gracefully** and continue with other images
5. **Return verified WordPress media IDs** for successful uploads

## Example Workflow

```python
def inteles_image_curator_enhanced(article_path):
    images = plan_images_for_article(article_path)
    uploaded_images = []

    for img in images:
        # Process and optimize image (save to local path)
        local_path = process_and_save_image(img)

        # Upload to WordPress with ALL parameters
        upload_result = upload_image_to_wordpress(
            image_path=local_path,
            title=img["title"],
            alt_text=img["alt_text"],
            caption=img["caption"]
        )

        if upload_result["success"]:
            uploaded_images.append({
                "media_id": upload_result["media_id"],
                "wordpress_url": upload_result["wordpress_url"],
                "alt_text": upload_result["alt_text"],
                "caption": upload_result["caption"],
                "placement": img["placement"],
                "priority": img["priority"]
            })
            print(f"✅ Uploaded: {img['title']} (ID: {upload_result['media_id']})")
        else:
            print(f"❌ Upload failed: {upload_result['error']}")

    return {
        "success": True if uploaded_images else False,
        "images": uploaded_images,
        "total_uploaded": len(uploaded_images),
        "verification_method": "wordpress_media_ids"
    }
```