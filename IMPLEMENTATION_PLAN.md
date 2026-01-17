# Product Image Search Automation - Implementation Plan

## Project Overview

This document outlines the technical implementation plan for building a complete Product Image Search Automation system that allows users to automatically search for product images, review them, and approve or retry until the correct image is found.

### Core Objectives

- Automate product image discovery from product names
- Implement quality filtering for images (product-only, clean background, high quality)
- Provide a manual approval workflow with retry capability
- Integrate with Google Sheets for data management
- Export results to CSV format

---

## System Architecture

```
┌─────────────────┐
│  Google Sheets  │
│  (Input Data)   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│         FastAPI Backend                 │
│  ┌─────────────────────────────────┐   │
│  │  Image Search Service           │   │
│  │  - Crawlee Integration          │   │
│  │  - Quality Validation           │   │
│  │  - URL Extraction               │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │  Workflow Management            │   │
│  │  - Status Tracking              │   │
│  │  - Retry Logic                  │   │
│  │  - Attempt Counter              │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │  Google Sheets Integration      │   │
│  │  - Read Products                │   │
│  │  - Update Status                │   │
│  │  - Save Results                 │   │
│  └─────────────────────────────────┘   │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│         React Frontend                  │
│  ┌─────────────────────────────────┐   │
│  │  Product Review Dashboard       │   │
│  │  - Image Preview                │   │
│  │  │  - Approve Button             │   │
│  │  - Retry Button                 │   │
│  │  - Attempt Counter              │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │  Batch Processing View          │   │
│  │  - Product List                 │   │
│  │  - Status Overview              │   │
│  │  - CSV Export                   │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

---

## Technology Stack

### Backend
- **FastAPI** - REST API framework
- **Crawlee** - Web scraping and image search
- **gspread** - Google Sheets API integration
- **Pillow** - Image validation and processing
- **Pydantic** - Data validation

### Frontend
- **React** - UI framework
- **Axios** - HTTP client
- **TailwindCSS** or **Vanilla CSS** - Styling
- **React Query** - Data fetching and caching

### Infrastructure
- **Python 3.10+** - Runtime environment
- **Node.js 18+** - Frontend build tools
- **Google Cloud Console** - Sheets API credentials

---

## Proposed Changes

### Backend Components

#### [NEW] `backend/main.py`
FastAPI application entry point with CORS configuration and route registration.

#### [NEW] `backend/models.py`
Pydantic models for:
- `Product` - Product data structure
- `ImageSearchRequest` - Search request payload
- `ImageSearchResponse` - Search result with URL and metadata
- `ApprovalRequest` - Approval/retry action payload
- `ProductStatus` - Status enumeration (pending, approved, retry)

#### [NEW] `backend/services/image_search.py`
Image search service using Crawlee:
- Search Google Images or other sources
- Filter images by criteria (background, quality, format)
- Extract direct image URLs (.jpg, .png)
- Validate image accessibility and quality
- Return ranked results

#### [NEW] `backend/services/image_validator.py`
Image quality validation service:
- Download and analyze images
- Detect backgrounds (white/clean vs complex)
- Check for watermarks and text overlays
- Verify resolution and clarity
- Detect people in images (optional ML model)

#### [NEW] `backend/services/sheets_service.py`
Google Sheets integration:
- Authenticate with service account
- Read product names from specified sheet
- Update image URLs and status
- Track attempt counts
- Handle batch operations

#### [NEW] `backend/services/workflow_service.py`
Workflow management:
- Process product queue
- Handle approval/retry logic
- Track attempt limits
- Update product status
- Generate CSV exports

#### [NEW] `backend/routes/products.py`
API endpoints:
- `GET /api/products` - List all products with status
- `GET /api/products/{id}` - Get single product details
- `POST /api/products/search` - Trigger image search for product
- `POST /api/products/{id}/approve` - Approve image
- `POST /api/products/{id}/retry` - Retry image search
- `POST /api/products/batch-search` - Process multiple products
- `GET /api/products/export` - Export to CSV

#### [NEW] `backend/config.py`
Configuration management:
- Google Sheets credentials path
- Search API settings
- Image quality thresholds
- Retry limits
- CORS origins

---

### Frontend Components

#### [NEW] `frontend/src/App.jsx`
Main application component with routing:
- Dashboard view
- Product review view
- Settings view

#### [NEW] `frontend/src/components/ProductCard.jsx`
Individual product card displaying:
- Product name
- Current image preview
- Status badge (pending/approved/retry)
- Attempt counter
- Approve/Retry buttons
- Loading states

#### [NEW] `frontend/src/components/ProductList.jsx`
Product list view with:
- Filterable product grid
- Status filters (all/pending/approved)
- Search functionality
- Batch actions

#### [NEW] `frontend/src/components/Dashboard.jsx`
Dashboard overview:
- Statistics (total, pending, approved)
- Recent activity
- Batch processing controls
- CSV export button

#### [NEW] `frontend/src/components/ImagePreview.jsx`
Image preview component:
- Large image display
- Zoom functionality
- Quality indicators
- Direct URL display

#### [NEW] `frontend/src/services/api.js`
API client service:
- Axios instance with base URL
- API method wrappers
- Error handling
- Request/response interceptors

#### [NEW] `frontend/src/hooks/useProducts.js`
React Query hooks:
- `useProducts` - Fetch product list
- `useProduct` - Fetch single product
- `useApproveProduct` - Approve mutation
- `useRetryProduct` - Retry mutation
- `useSearchProduct` - Search mutation

---

### Crawlee Integration

#### [NEW] `backend/crawlers/image_crawler.py`
Crawlee-based image crawler:
- Configure Playwright/Puppeteer crawler
- Search Google Images with product name
- Extract image URLs from search results
- Filter by file extension (.jpg, .png)
- Handle pagination if needed
- Respect rate limits

#### [NEW] `backend/crawlers/quality_filter.py`
Image quality filtering logic:
- Download candidate images
- Check image dimensions (min resolution)
- Analyze background complexity
- Detect text/watermarks using OCR
- Score images by quality criteria
- Return top candidates

---

### Configuration Files

#### [NEW] `backend/requirements.txt`
Python dependencies:
```
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
crawlee>=0.3.0
playwright>=1.40.0
gspread>=5.12.0
google-auth>=2.23.0
pillow>=10.1.0
pydantic>=2.5.0
python-multipart>=0.0.6
aiofiles>=23.2.1
```

#### [NEW] `frontend/package.json`
Node.js dependencies:
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.6.0",
    "@tanstack/react-query": "^5.8.0",
    "react-router-dom": "^6.20.0"
  }
}
```

#### [NEW] `.env.example`
Environment variables template:
```
GOOGLE_SHEETS_CREDENTIALS_PATH=./credentials.json
GOOGLE_SHEET_ID=your_sheet_id_here
MAX_RETRY_ATTEMPTS=3
IMAGE_MIN_WIDTH=800
IMAGE_MIN_HEIGHT=800
BACKEND_PORT=8000
FRONTEND_PORT=3000
```

#### [NEW] `README.md`
Project documentation:
- Setup instructions
- Google Sheets API setup guide
- Running the application
- API documentation
- Workflow explanation
- Troubleshooting

---

## Data Flow

### 1. Initial Setup
```
User → Google Sheets → Add product names
User → Backend → Configure credentials
User → Frontend → Start application
```

### 2. Image Search Flow
```
Frontend → POST /api/products/batch-search
Backend → Read products from Google Sheets
Backend → For each product:
  → Crawlee searches for images
  → Quality filter validates images
  → Best image URL selected
  → Update Sheets with URL (status: pending)
Backend → Return results to Frontend
Frontend → Display products for review
```

### 3. Approval Flow
```
User → Review image in Frontend
User → Click "Approve"
Frontend → POST /api/products/{id}/approve
Backend → Update status to "approved" in Sheets
Backend → Return success
Frontend → Update UI
```

### 4. Retry Flow
```
User → Review image in Frontend
User → Click "Retry"
Frontend → POST /api/products/{id}/retry
Backend → Increment attempt counter
Backend → Search for new image (exclude previous)
Backend → Update Sheets with new URL
Backend → Return new image
Frontend → Display new image for review
```

### 5. Export Flow
```
User → Click "Export CSV"
Frontend → GET /api/products/export
Backend → Generate CSV with approved products
Backend → Return CSV file
Frontend → Download CSV
```

---

## Google Sheets Structure

### Sheet Name: `Products`

| Column | Field Name | Type | Description |
|--------|-----------|------|-------------|
| A | product_name | String | Product name to search |
| B | image_url | String | Direct image URL (.jpg/.png) |
| C | status | String | pending / approved / retry |
| D | attempts | Integer | Number of search attempts |
| E | notes | String | Optional notes or error messages |
| F | created_at | Timestamp | When product was added |
| G | updated_at | Timestamp | Last update time |

---

## Image Quality Criteria

### Required Criteria
1. **Product-only image** - No people in the image
2. **Clean background** - White or solid color background preferred
3. **High quality** - Minimum 800x800 pixels
4. **Clear resolution** - Sharp, not blurry
5. **No watermarks** - No text overlays or promotional content
6. **Direct URL** - Must end with .jpg or .png

### Validation Process
1. Download image to temporary location
2. Check file format and extension
3. Verify dimensions meet minimum requirements
4. Analyze background complexity (edge detection)
5. Run OCR to detect text overlays
6. (Optional) Use ML model to detect people
7. Calculate quality score (0-100)
8. Return pass/fail with score

---

## API Endpoints

### Products

#### `GET /api/products`
List all products with pagination and filtering.

**Query Parameters:**
- `status` - Filter by status (pending/approved/retry)
- `page` - Page number (default: 1)
- `limit` - Items per page (default: 20)

**Response:**
```json
{
  "products": [
    {
      "id": 1,
      "product_name": "Wireless Mouse",
      "image_url": "https://example.com/image.jpg",
      "status": "pending",
      "attempts": 1,
      "notes": "",
      "created_at": "2026-01-17T21:50:00Z",
      "updated_at": "2026-01-17T21:50:00Z"
    }
  ],
  "total": 100,
  "page": 1,
  "pages": 5
}
```

#### `POST /api/products/search`
Search for image for a specific product.

**Request:**
```json
{
  "product_id": 1,
  "exclude_urls": ["https://previous-image.jpg"]
}
```

**Response:**
```json
{
  "product_id": 1,
  "image_url": "https://example.com/new-image.jpg",
  "quality_score": 85,
  "status": "pending"
}
```

#### `POST /api/products/{id}/approve`
Approve the current image for a product.

**Response:**
```json
{
  "product_id": 1,
  "status": "approved",
  "message": "Image approved successfully"
}
```

#### `POST /api/products/{id}/retry`
Retry image search for a product.

**Response:**
```json
{
  "product_id": 1,
  "image_url": "https://example.com/retry-image.jpg",
  "attempts": 2,
  "status": "pending"
}
```

#### `POST /api/products/batch-search`
Process multiple products at once.

**Request:**
```json
{
  "product_ids": [1, 2, 3, 4, 5],
  "max_concurrent": 3
}
```

**Response:**
```json
{
  "processed": 5,
  "successful": 4,
  "failed": 1,
  "results": [...]
}
```

#### `GET /api/products/export`
Export approved products to CSV.

**Query Parameters:**
- `status` - Filter by status (default: approved)

**Response:** CSV file download

---

## Workflow Logic

### Automatic Search Process

```python
def process_product(product_name, attempt=1, max_attempts=3):
    """
    Process a single product through the image search workflow.
    """
    # 1. Search for images
    images = search_images(product_name)
    
    # 2. Filter by quality criteria
    valid_images = []
    for image_url in images:
        if validate_image_quality(image_url):
            valid_images.append(image_url)
    
    # 3. Select best image
    if valid_images:
        best_image = valid_images[0]
        
        # 4. Update Google Sheets
        update_sheet(
            product_name=product_name,
            image_url=best_image,
            status="pending",
            attempts=attempt
        )
        
        return {
            "success": True,
            "image_url": best_image,
            "status": "pending"
        }
    else:
        # No valid images found
        if attempt < max_attempts:
            # Retry with different search terms
            return process_product(
                product_name, 
                attempt=attempt+1, 
                max_attempts=max_attempts
            )
        else:
            # Max attempts reached
            update_sheet(
                product_name=product_name,
                image_url="",
                status="retry",
                attempts=attempt,
                notes="No valid images found after max attempts"
            )
            return {
                "success": False,
                "message": "Max attempts reached"
            }
```

### Manual Review Process

```
1. User opens frontend dashboard
2. System displays products with status "pending"
3. For each product:
   a. User views image preview
   b. User decides:
      - APPROVE → Status changes to "approved"
      - RETRY → System searches again, increments attempts
4. Approved products can be exported to CSV
```

---

## Setup Instructions

### 1. Google Sheets API Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing
3. Enable Google Sheets API
4. Create service account credentials
5. Download JSON credentials file
6. Share your Google Sheet with service account email
7. Save credentials as `credentials.json` in project root

### 2. Backend Setup

```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
playwright install  # Install browser for Crawlee
cp .env.example .env  # Configure environment variables
python main.py  # Start FastAPI server
```

### 3. Frontend Setup

```bash
cd frontend
npm install
npm run dev  # Start development server
```

### 4. Access Application

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- API Docs: http://localhost:8000/docs

---

## Verification Plan

### Automated Tests

#### Backend Tests
```bash
cd backend
pytest tests/test_image_search.py
pytest tests/test_quality_validator.py
pytest tests/test_sheets_service.py
pytest tests/test_workflow.py
```

**Test Coverage:**
- Image search returns valid URLs
- Quality validation correctly filters images
- Google Sheets integration reads/writes correctly
- Workflow handles approval/retry logic
- API endpoints return expected responses

#### Frontend Tests
```bash
cd frontend
npm test
```

**Test Coverage:**
- Components render correctly
- API calls are made with correct parameters
- Approve/Retry buttons trigger correct actions
- CSV export downloads file

### Manual Verification

1. **Image Search Quality**
   - Verify searched images meet quality criteria
   - Check that backgrounds are clean/white
   - Confirm no watermarks or text overlays
   - Validate image URLs end with .jpg or .png

2. **Approval Workflow**
   - Test approve button updates status
   - Verify retry button searches for new image
   - Check attempt counter increments correctly
   - Confirm max attempts limit works

3. **Google Sheets Integration**
   - Verify products are read from sheet
   - Check status updates reflect in sheet
   - Confirm timestamps are recorded
   - Validate CSV export matches sheet data

4. **UI/UX Testing**
   - Test responsive design on different screens
   - Verify loading states display correctly
   - Check error messages are user-friendly
   - Confirm image previews load properly

---

## Future Expansion Recommendations

### Phase 2 Enhancements

1. **Advanced Image Search**
   - Multiple search engines (Bing, DuckDuckGo)
   - Reverse image search for verification
   - AI-powered image generation as fallback

2. **Improved Quality Detection**
   - ML model for background detection
   - People detection using computer vision
   - Brand logo detection
   - Image similarity scoring

3. **Batch Operations**
   - Bulk approve/retry
   - Scheduled automatic processing
   - Priority queue for urgent products

4. **Analytics & Reporting**
   - Success rate dashboard
   - Search performance metrics
   - Quality score distribution
   - Time-to-approval tracking

5. **User Management**
   - Multi-user support
   - Role-based permissions
   - Approval workflows with multiple reviewers

6. **Integration Enhancements**
   - Direct upload to cloud storage (S3, GCS)
   - Shopify/WooCommerce integration
   - Webhook notifications
   - Slack/Email alerts

---

## Risk Mitigation

### Potential Issues

1. **Rate Limiting**
   - **Risk:** Search engines may block requests
   - **Mitigation:** Implement delays, use proxies, rotate user agents

2. **Image Quality Variability**
   - **Risk:** Automated quality detection may miss issues
   - **Mitigation:** Manual review step, adjustable quality thresholds

3. **API Quota Limits**
   - **Risk:** Google Sheets API has usage limits
   - **Mitigation:** Batch operations, caching, local database option

4. **Image URL Expiration**
   - **Risk:** Direct URLs may become invalid over time
   - **Mitigation:** Download and host images locally, periodic validation

---

## Project Timeline Estimate

### Week 1: Backend Foundation
- Setup FastAPI project structure
- Implement Google Sheets integration
- Create data models and database schema

### Week 2: Image Search & Validation
- Integrate Crawlee for web scraping
- Implement image quality validation
- Build search and filtering logic

### Week 3: API & Workflow
- Create REST API endpoints
- Implement approval/retry workflow
- Add CSV export functionality

### Week 4: Frontend Development
- Setup React application
- Build product list and card components
- Implement approval interface

### Week 5: Integration & Testing
- Connect frontend to backend
- Write automated tests
- Perform manual testing

### Week 6: Polish & Documentation
- Fix bugs and edge cases
- Write comprehensive documentation
- Prepare deployment guide

---

## Success Criteria

The project will be considered successful when:

1. ✅ System can automatically search for product images
2. ✅ Images meet all quality criteria (clean background, no watermarks, etc.)
3. ✅ Manual approval workflow functions correctly
4. ✅ Retry mechanism finds alternative images
5. ✅ Google Sheets integration reads and updates data
6. ✅ CSV export generates correct output
7. ✅ Frontend provides intuitive user experience
8. ✅ System is stable and handles errors gracefully
9. ✅ Documentation is complete and clear
10. ✅ All automated tests pass

---

## Conclusion

This implementation plan provides a comprehensive roadmap for building the Product Image Search Automation system. The modular architecture allows for incremental development and future enhancements while maintaining stability and reliability.

The combination of automated search with manual approval ensures both efficiency and quality control, meeting the core project requirements.
