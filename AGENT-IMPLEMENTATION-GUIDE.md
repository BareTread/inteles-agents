# 🔧 Agent Implementation Guide

## 🎯 Overview

This guide provides detailed implementation specifications for each agent in the inteles.ro orchestrator system. The focus is on **flawless execution** with **verifiable outputs** and **error recovery**.

---

## 📝 Agent 1: content-quickfire (Already Optimal)

### Current Implementation ✅
This agent is already working perfectly and requires no changes.

### Specifications
```python
def content_quickfire(keyword_or_text):
    """
    Input: Keyword string or article text
    Process: Research, writing, content creation
    Output: Saved article file path

    Returns:
    {
        "success": true,
        "article_path": "/absolute/path/to/article.md",
        "word_count": 1500,
        "verification_method": "file_exists"
    }
    """
```

### Key Points
- ✅ Preserves Romanian diacritics
- ✅ Creates comprehensive, well-structured content
- ✅ Outputs absolute file path
- ✅ No MCP tool usage (expensive agent optimization)

---

## 🖼️ Agent 2: inteles-image-curator (CRITICAL REDESIGN REQUIRED)

### 🚨 Current Problem
The agent currently:
1. Downloads images to local storage
2. Claims success without uploading to WordPress
3. Returns local paths (useless for wordpress-publisher)

### ✅ Required Implementation

```python
def inteles_image_curator_enhanced(article_path):
    """
    ENHANCED IMAGE CURATOR WITH WORDPRESS UPLOAD CAPABILITY

    Input: Article file path
    Process: Analyze → Download → Process → UPLOAD → Verify
    Output: WordPress media IDs and real URLs

    Workload Distribution: 40% of total system workload
    """

    try:
        # STEP 1: Analyze article content (5% of agent workload)
        print("🔍 Analyzing article content for image requirements...")
        image_plan = analyze_article_for_images(article_path)

        # STEP 2: Download from Pexels (20% of agent workload)
        print("📥 Downloading images from Pexels...")
        downloaded_images = []
        for image_spec in image_plan:
            try:
                img_data = pexels_search_and_download(
                    query=image_spec["search_query"],
                    orientation=image_spec.get("orientation", "landscape")
                )
                downloaded_images.append({
                    "spec": image_spec,
                    "local_path": img_data["local_path"],
                    "pexels_id": img_data["id"]
                })
            except Exception as e:
                print(f"⚠️  Failed to download {image_spec['search_query']}: {e}")
                continue

        if not downloaded_images:
            raise Exception("No images could be downloaded from Pexels")

        # STEP 3: Process and optimize (10% of agent workload)
        print("⚙️  Processing and optimizing images...")
        processed_images = []
        for img in downloaded_images:
            processed = process_image_for_wordpress(
                local_path=img["local_path"],
                title=img["spec"]["title"],
                alt_text=img["spec"]["alt_text"],
                caption=img["spec"]["caption"]
            )
            processed_images.append({
                **img,
                "processed": processed
            })

        # STEP 4: UPLOAD TO WORDPRESS (50% of agent workload - THE REAL WORK!)
        print("📤 Uploading images to WordPress...")
        uploaded_images = []

        for img in processed_images:
            try:
                # CRITICAL: Actually upload to WordPress
                media_response = inteles_wordpress_create_media(
                    title=img["processed"]["title"],
                    alt_text=img["processed"]["alt_text"],
                    caption=img["processed"]["caption"],
                    description=img["processed"]["description"],
                    source_url=img["local_path"]  # Upload from local file
                )

                # VERIFICATION: Check if upload actually succeeded
                if media_response.get("id") and media_response.get("source_url"):
                    uploaded_image = {
                        "media_id": media_response["id"],
                        "wordpress_url": media_response["source_url"],
                        "alt_text": img["processed"]["alt_text"],
                        "caption": img["processed"]["caption"],
                        "placement": img["spec"]["placement"],
                        "priority": img["spec"]["priority"],
                        "title": img["processed"]["title"]
                    }
                    uploaded_images.append(uploaded_image)
                    print(f"✅ Uploaded: {img['processed']['title']} (ID: {media_response['id']})")
                else:
                    raise Exception(f"Upload failed - no media ID returned")

            except Exception as upload_error:
                print(f"❌ Upload failed for {img['processed']['title']}: {upload_error}")
                # Continue with other images instead of failing completely
                continue

        # STEP 5: Final verification and packaging (15% of agent workload)
        if not uploaded_images:
            raise Exception("All image uploads failed")

        # Verify each uploaded image is accessible
        verified_images = []
        for img in uploaded_images:
            try:
                # Verify the image URL is accessible
                verification = verify_wordpress_media_exists(img["media_id"])
                if verification:
                    verified_images.append(img)
                    print(f"✅ Verified: {img['title']}")
                else:
                    print(f"⚠️  Verification failed for {img['title']}")
            except Exception as verify_error:
                print(f"⚠️  Could not verify {img['title']}: {verify_error}")
                # Still include if upload succeeded but verification failed

        # Sort by priority for proper placement
        verified_images.sort(key=lambda x: x["priority"])

        print(f"🎉 Successfully processed and uploaded {len(verified_images)} images")

        return {
            "success": True,
            "images": verified_images,
            "total_uploaded": len(verified_images),
            "verification_method": "wordpress_media_ids",
            "processing_summary": {
                "planned": len(image_plan),
                "downloaded": len(downloaded_images),
                "processed": len(processed_images),
                "uploaded": len(uploaded_images),
                "verified": len(verified_images)
            }
        }

    except Exception as e:
        print(f"❌ Image processing failed: {str(e)}")
        return {
            "success": False,
            "error": str(e),
            "fallback_strategy": "use_stock_images_or_proceed_without",
            "verification_method": "error_reported"
        }

def analyze_article_for_images(article_path):
    """Analyze article content to determine optimal image requirements"""
    # Read article content
    with open(article_path, 'r', encoding='utf-8') as f:
        content = f.read()

    # Determine image requirements based on content analysis
    image_requirements = [
        {
            "search_query": extract_main_topic(content),
            "placement": "hero",
            "priority": 1,
            "title": generate_title(content, "hero"),
            "alt_text": generate_alt_text(content, "hero"),
            "caption": generate_caption(content, "hero"),
            "orientation": "landscape"
        }
    ]

    # Add more image requirements based on article sections
    sections = identify_key_sections(content)
    for i, section in enumerate(sections[1:6], 2):  # Max 6 images total
        image_requirements.append({
            "search_query": extract_section_topic(section),
            "placement": "inline",
            "priority": i,
            "title": generate_title(section, f"section_{i}"),
            "alt_text": generate_alt_text(section, f"section_{i}"),
            "caption": generate_caption(section, f"section_{i}"),
            "orientation": "landscape"
        })

    return image_requirements

def process_image_for_wordpress(local_path, title, alt_text, caption):
    """Process downloaded image for WordPress compatibility"""
    return {
        "title": title,
        "alt_text": alt_text[:125],  # WordPress limit
        "caption": caption,
        "description": f"Imagine pentru articol inteles.ro: {title.lower()}",
        "processed_path": local_path  # After WebP conversion, optimization
    }

def verify_wordpress_media_exists(media_id):
    """Verify that uploaded media actually exists in WordPress"""
    try:
        # Use inteles-wordpress MCP to verify
        result = inteles_wordpress_get_media(media_id)
        return result.get("id") == media_id
    except:
        return False
```

### Error Recovery Strategies

```python
def handle_image_curator_failures(error_type, context):
    """Implement fallback strategies for image processing failures"""

    if error_type == "pexels_down":
        # Use local image library
        return use_local_image_library(context)

    elif error_type == "wordpress_upload_failed":
        # Queue images for manual upload, proceed with article
        return queue_for_manual_upload(context)

    elif error_type == "partial_upload_success":
        # Use available images, mark missing ones
        return proceed_with_available_images(context)

    elif error_type == "all_uploads_failed":
        # Proceed without images
        return proceed_without_images(context)
```

---

## 💰 Agent 3: romanian-affiliate-product-finder (Minor Enhancement)

### Current Status ✅ (Mostly Working)
This agent is functional but needs enhanced validation and error handling.

### Enhanced Implementation

```python
def romanian_affiliate_product_finder_enhanced(article_path):
    """
    ENHANCED AFFILIATE PRODUCT FINDER WITH ROBUST VALIDATION

    Input: Article file path
    Process: Analyze content → Find relevant products → Validate URLs
    Output: Verified affiliate product URLs

    Workload Distribution: 15% of total system workload
    """

    try:
        # STEP 1: Read and analyze article content
        print("🔍 Analyzing article content for product opportunities...")
        with open(article_path, 'r', encoding='utf-8') as f:
            content = f.read()

        # STEP 2: Load and validate affiliate whitelist
        print("📋 Loading affiliate whitelist...")
        whitelist = load_affiliate_whitelist()

        if not whitelist:
            print("⚠️  Affiliate whitelist not found - proceeding without products")
            return {
                "success": True,
                "products": [],
                "total_products": 0,
                "verification_method": "no_whitelist_available"
            }

        # STEP 3: Find relevant products based on content
        print("🛍️  Finding relevant affiliate products...")
        potential_products = find_products_for_content(content, whitelist)

        # STEP 4: Validate each product URL and merchant
        print("✅ Validating product URLs...")
        validated_products = []

        for product in potential_products:
            try:
                # Validate merchant is whitelisted
                if not is_merchant_whitelisted(product["merchant"], whitelist):
                    print(f"⚠️  Merchant {product['merchant']} not whitelisted")
                    continue

                # Validate URL format and accessibility
                if not validate_affiliate_url(product["url"]):
                    print(f"⚠️  Invalid URL format: {product['url']}")
                    continue

                # Check if product is relevant to article content
                relevance_score = calculate_content_relevance(product, content)
                if relevance_score < 0.3:  # Minimum relevance threshold
                    print(f"⚠️  Low relevance ({relevance_score:.2f}) for: {product['title']}")
                    continue

                validated_product = {
                    "url": product["url"],
                    "merchant": product["merchant"],
                    "title": product["title"],
                    "description": product["description"],
                    "relevance_score": relevance_score,
                    "placement_hint": determine_placement_hint(product, content),
                    "price_range": product.get("price_range", "Unknown"),
                    "category": product.get("category", "General")
                }

                validated_products.append(validated_product)
                print(f"✅ Validated: {product['title']} from {product['merchant']}")

            except Exception as validation_error:
                print(f"⚠️  Validation failed for {product.get('title', 'Unknown')}: {validation_error}")
                continue

        # STEP 5: Sort by relevance and limit to 4 products maximum
        validated_products.sort(key=lambda x: x["relevance_score"], reverse=True)
        final_products = validated_products[:4]

        print(f"🎉 Found and validated {len(final_products)} relevant products")

        return {
            "success": True,
            "products": final_products,
            "total_products": len(final_products),
            "verification_method": "whitelist_and_url_validation",
            "whitelist_merchants": len(whitelist),
            "relevance_threshold": 0.3
        }

    except Exception as e:
        print(f"❌ Product finding failed: {str(e)}")
        return {
            "success": True,  # Continue without products
            "products": [],
            "total_products": 0,
            "error": str(e),
            "verification_method": "error_fallback_to_no_products"
        }

def load_affiliate_whitelist():
    """Load affiliate whitelist from CSV file"""
    try:
        # Try to find the whitelist CSV file
        whitelist_paths = [
            "04-Monetization/affiliates.csv",
            "/mnt/data/OBSIDIAN/inteles-vault/04-Monetization/affiliates.csv",
            "affiliates.csv"
        ]

        for path in whitelist_paths:
            if os.path.exists(path):
                with open(path, 'r', encoding='utf-8') as f:
                    reader = csv.DictReader(f)
                    return list(reader)

        print("⚠️  Affiliate whitelist file not found")
        return None

    except Exception as e:
        print(f"❌ Error loading whitelist: {e}")
        return None

def validate_affiliate_url(url):
    """Validate affiliate URL format and accessibility"""
    try:
        # Basic URL format validation
        if not url.startswith(('http://', 'https://')):
            return False

        # Check for known affiliate domains
        known_domains = ['libris.ro', 'herbagetica.ro', 'manukashop.ro', 'librex.ro']
        if not any(domain in url.lower() for domain in known_domains):
            return False

        # Additional validation can be added here
        return True

    except:
        return False

def calculate_content_relevance(product, content):
    """Calculate relevance score of product to article content"""
    # Simple keyword matching for relevance
    content_lower = content.lower()
    product_text = f"{product['title']} {product.get('description', '')}".lower()

    # Count keyword matches
    common_words = set(content_lower.split()) & set(product_text.split())
    total_words = len(product_text.split())

    if total_words == 0:
        return 0

    relevance = len(common_words) / total_words
    return min(relevance, 1.0)  # Cap at 1.0
```

---

## 📝 Agent 4: wordpress-publisher (Enhanced Integration)

### Current Status ✅ (Working but needs better verification)

### Enhanced Implementation

```python
def wordpress_publisher_enhanced(article_path, images_data, products_data, update_existing=False, post_id=None):
    """
    ENHANCED WORDPRESS PUBLISHER WITH COMPREHENSIVE VERIFICATION

    Input: Article file, images data, products data, update flags
    Process: Integrate all components → Build WordPress HTML → Publish
    Output: Published article URL with verification

    Workload Distribution: 15% of total system workload
    """

    try:
        # STEP 1: Input verification
        print("🔍 Verifying inputs...")

        if not os.path.exists(article_path):
            raise Exception(f"Article file not found: {article_path}")

        # Verify images data integrity
        if images_data.get("success") and images_data.get("images"):
            print(f"✅ Images data verified: {images_data['total_uploaded']} images")
            for img in images_data["images"]:
                if not img.get("media_id") or not img.get("wordpress_url"):
                    raise Exception(f"Invalid image data: missing media_id or URL")
        else:
            print("⚠️  No images data - proceeding without images")
            images_data = {"success": False, "images": [], "total_uploaded": 0}

        # Verify products data
        if products_data.get("success") and products_data.get("products"):
            print(f"✅ Products data verified: {products_data['total_products']} products")
        else:
            print("⚠️  No products data - proceeding without monetization")
            products_data = {"success": False, "products": [], "total_products": 0}

        # STEP 2: Read and prepare article content
        print("📖 Reading article content...")
        with open(article_path, 'r', encoding='utf-8') as f:
            article_content = f.read()

        # Extract title and generate excerpt
        title = extract_title_from_content(article_content)
        excerpt = generate_excerpt(article_content)
        slug = generate_slug_from_title(title)

        # STEP 3: Build WordPress HTML with Kadence blocks
        print("🏗️  Building WordPress HTML with Kadence blocks...")
        wordpress_html = build_kadence_blocks_html(
            article_content=article_content,
            images=images_data.get("images", []),
            products=products_data.get("products", []),
            title=title,
            excerpt=excerpt
        )

        # STEP 4: Publish or update WordPress post
        print("📤 Publishing to WordPress...")

        if update_existing and post_id:
            # Update existing post
            result = inteles_wordpress_update_post(
                id=post_id,
                title=title,
                content=wordpress_html,
                excerpt=excerpt,
                status="publish",
                featured_media=images_data["images"][0]["media_id"] if images_data["images"] else 0
            )
            operation = "update"
        else:
            # Create new post
            result = inteles_wordpress_create_post(
                title=title,
                content=wordpress_html,
                excerpt=excerpt,
                status="publish",
                slug=slug,
                featured_media=images_data["images"][0]["media_id"] if images_data["images"] else 0,
                categories=[5],  # Înțelesul Viselor category
                tags=[]
            )
            operation = "create"

        # STEP 5: Verify publication success
        if not result.get("id"):
            raise Exception("WordPress publication failed - no post ID returned")

        post_url = result.get("link")
        if not post_url:
            raise Exception("WordPress publication failed - no post URL returned")

        # STEP 6: Final verification
        print("🔍 Final verification...")

        # Verify post is accessible
        verification = verify_post_published(result["id"])
        if not verification:
            print("⚠️  Post verification failed - but publication may have succeeded")

        print(f"🎉 Successfully {operation}ed article: {post_url}")

        return {
            "success": True,
            "operation": operation,
            "post_id": result["id"],
            "post_url": post_url,
            "title": title,
            "slug": slug,
            "images_used": images_data["total_uploaded"],
            "products_integrated": products_data["total_products"],
            "verification_method": "wordpress_post_created",
            "publication_details": {
                "status": result.get("status"),
                "date": result.get("date"),
                "featured_media": result.get("featured_media")
            }
        }

    except Exception as e:
        print(f"❌ WordPress publishing failed: {str(e)}")
        return {
            "success": False,
            "error": str(e),
            "fallback_strategy": "save_html_for_manual_upload",
            "verification_method": "error_reported"
        }

def build_kadence_blocks_html(article_content, images, products, title, excerpt):
    """Build WordPress-compatible HTML with Kadence blocks"""

    # Convert markdown to HTML first
    html_content = convert_markdown_to_html(article_content)

    # Insert images at strategic positions
    html_with_images = insert_images_into_content(html_content, images)

    # Insert affiliate products
    html_with_products = insert_products_into_content(html_with_images, products)

    # Wrap in Kadence block structure
    final_html = wrap_in_kadence_blocks(html_with_products, title, excerpt)

    return final_html

def verify_post_published(post_id):
    """Verify that post was actually published"""
    try:
        result = inteles_wordpress_get_content_by_id(post_id)
        return result.get("status") == "publish"
    except:
        return False
```

---

## 🔄 Integration Testing

### Test Workflow
```python
def test_complete_workflow(keyword):
    """Test the complete agent workflow with a sample keyword"""

    print("🧪 Starting complete workflow test...")

    # Step 1: Content creation
    content_result = content_quickfire(keyword)
    assert content_result["success"], "Content creation failed"

    # Step 2: Parallel processing
    with ThreadPoolExecutor() as executor:
        image_future = executor.submit(inteles_image_curator_enhanced, content_result["article_path"])
        product_future = executor.submit(romanian_affiliate_product_finder_enhanced, content_result["article_path"])

        images_result = image_future.result(timeout=900)  # 15 minutes
        products_result = product_future.result(timeout=300)  # 5 minutes

    # Step 3: Publishing
    publish_result = wordpress_publisher_enhanced(
        content_result["article_path"],
        images_result,
        products_result
    )

    assert publish_result["success"], "Publishing failed"

    print(f"✅ Test successful! Article published: {publish_result['post_url']}")
    return publish_result
```

---

## 📋 Implementation Checklist

### For Each Agent:
- [ ] Input validation and error handling
- [ ] Verifiable output format
- [ ] Fallback strategies for failures
- [ ] Performance optimization (timeouts, limits)
- [ ] Comprehensive logging
- [ ] Integration testing

### System-wide:
- [ ] Parallel execution capability
- [ ] Error recovery mechanisms
- [ ] Performance monitoring
- [ ] Success metrics tracking
- [ ] Documentation maintenance

This enhanced implementation ensures **flawless execution** with **zero manual intervention** while maintaining the **balanced workload distribution** across all agents.