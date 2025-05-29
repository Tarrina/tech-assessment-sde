# NearPharma

## Overview
Build a REST API service that helps users explore pharmacy locations and nearby services using Google Maps APIs. This project focuses on integrating with Google Maps APIs to provide location-based services for pharmacy discovery and route planning.

## Time Estimate
**3 hours**

## Tech Stack Requirements
- **Language**: Java (Spring Boot)
- **Database**: SQLite or PostgreSQL (for storing pharmacy data)
- **APIs**: Google Maps Distance Matrix API, Directions API, Places API
- **Documentation**: README with setup instructions

## Project Requirements

### Core Features

#### 1. Pharmacy Management
Create basic CRUD operations for pharmacy data:
- Store pharmacy information (name, address, coordinates, phone)
- Seed database with 5-10 sample pharmacies
- Basic validation for pharmacy data

#### 2. Distance Matrix Integration
**Endpoint**: `GET /api/pharmacies/distances`
- Accept user coordinates (lat, lng) as query parameters
- Calculate distances and travel times to all pharmacies using Google Distance Matrix API
- Return pharmacies sorted by travel time
- Support travel mode parameter (driving, walking, transit)

#### 3. Route Directions
**Endpoint**: `GET /api/pharmacies/{id}/directions`
- Get detailed directions from user location to specific pharmacy
- Use Google Directions API to return step-by-step directions
- Include duration, distance, and route polyline
- Support different travel modes

#### 4. Nearby Pharmacy Discovery
**Endpoint**: `GET /api/pharmacies/{id}/nearby`
- Find nearby pharmacies around a specific pharmacy using Google Places API
- Search specifically for `pharmacy` place type using Google Places API
- Return nearby pharmacy details: name, rating, address, distance from source pharmacy, contact info
- Support radius parameter (default 2km, max 10km for rural coverage)
- Support optional filtering by pharmacy chains (Apollo, MedPlus, etc.)
- Include operating hours and availability status when possible

#### 5. Location Search
**Endpoint**: `GET /api/places/search`
- Search for places by text query using Google Places API
- Support place type filtering using Google Places API types:
  - Use `pharmacy` type to find pharmacy locations
  - Support additional healthcare types: `hospital`, `doctor`, `dentist`
  - Use `establishment` type for general business searches
- Return basic place information: name, address, coordinates, place types, rating
- Support optional `types` parameter to restrict search results

### API Response Examples

#### Distance Matrix Response
```json
{
  "userLocation": {
    "lat": 23.0225,
    "lng": 72.5714
  },
  "pharmacies": [
    {
      "id": 1,
      "name": "Apollo Pharmacy - Satellite",
      "address": "Ground Floor, Iscon Mega Mall, Satellite Road, Ahmedabad, Gujarat 380015",
      "distance": "1.8 km",
      "duration": "6 mins",
      "coordinates": {
        "lat": 23.0258,
        "lng": 72.5698
      },
      "phone": "+91 79 4040 1234",
      "isOpen": true
    },
    {
      "id": 2,
      "name": "MedPlus Pharmacy - Vastrapur",
      "address": "Shop 12, Shivalik Plaza, Vastrapur Circle, Ahmedabad, Gujarat 380015",
      "distance": "2.4 km", 
      "duration": "9 mins",
      "coordinates": {
        "lat": 23.0395,
        "lng": 72.5659
      },
      "phone": "+91 79 2630 5678",
      "isOpen": true
    }
  ]
}
```

#### Nearby Pharmacy Response
```json
{
  "sourcePharmacy": {
    "id": 1,
    "name": "Apollo Pharmacy - Satellite",
    "address": "Ground Floor, Iscon Mega Mall, Satellite Road, Ahmedabad, Gujarat 380015"
  },
  "nearbyPharmacies": [
    {
      "placeId": "ChIJrTLr-GyuEmsRBfy61i59si0",
      "name": "Wellness Forever - Prahladnagar",
      "rating": 4.1,
      "address": "Shop 8-9, Crystal Plaza, Prahladnagar Garden Road, Ahmedabad, Gujarat 380015",
      "distanceFromSource": "850m",
      "phone": "+91 79 4896 7321",
      "isOpen": true,
      "operatingHours": "8:00 AM - 11:00 PM",
      "placeTypes": ["pharmacy", "health", "establishment"]
    },
    {
      "placeId": "ChIJN1t_tDeuEmsRAGlvNvjj8HE",
      "name": "Jan Aushadhi Store - SG Highway",
      "rating": 3.9,
      "address": "Near Prernatirth Derasar, SG Highway, Ahmedabad, Gujarat 380054",
      "distanceFromSource": "1.2km",
      "phone": "+91 79 2685 4321",
      "isOpen": true,
      "operatingHours": "9:00 AM - 9:00 PM",
      "placeTypes": ["pharmacy", "health", "establishment"]
    },
    {
      "placeId": "ChIJ39UEbGWuEmsR9UBGHqVx8lM",
      "name": "Netmeds Pharmacy - Bodakdev",
      "rating": 4.3,
      "address": "Ground Floor, Ganesh Genesis, Sarkhej-Gandhinagar Highway, Bodakdev, Ahmedabad, Gujarat 380054",
      "distanceFromSource": "1.5km",
      "phone": "+91 79 4567 8901",
      "isOpen": false,
      "operatingHours": "9:00 AM - 10:00 PM",
      "placeTypes": ["pharmacy", "health", "establishment"]
    }
  ],
  "searchCriteria": {
    "radius": "2000m",
    "placeType": "pharmacy",
    "includeChains": ["Apollo", "MedPlus", "Wellness Forever", "Jan Aushadhi", "Netmeds"]
  }
}
```

## Technical Requirements

### 1. Environment Setup
- Use environment variables for Google Maps API key
- Proper error handling for missing API credentials

### 2. API Integration
- Implement proper rate limiting for external API calls
- Cache Google Maps API responses for 5 minutes to avoid redundant calls
- Handle API errors gracefully with meaningful error messages

#### Google Places API Integration Guidelines
- **Place Types**: Focus primarily on `pharmacy` place type for nearby discovery
- **Search Requests**: Use Nearby Search API with `pharmacy` type and appropriate radius (this should be set in your .env file)
- **Response Handling**: Parse pharmacy-specific data including operating hours, contact info
- **Rural Coverage**: Support larger search radius (up to 10km) for rural pharmacy discovery

### 3. Data Validation
- Validate coordinates (latitude: -90 to 90, longitude: -180 to 180)
- Validate travel modes (driving, walking, transit, bicycling)
- Validate radius parameter (500m to 10000m for rural coverage)
- Validate pharmacy chain filters against supported chains:
  - Supported chains: Apollo, MedPlus, Wellness Forever, Jan Aushadhi, Netmeds, 1mg
  - Validate place type is specifically `pharmacy` for nearby discovery

### 4. Error Handling
- Return appropriate HTTP status codes
- Provide clear error messages for invalid requests
- Handle Google Maps API quota exceeded scenarios

## Database Schema

### Pharmacies Table
```sql
CREATE TABLE pharmacies (
    id INTEGER PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address TEXT NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    phone VARCHAR(20),
    chain VARCHAR(100),
    pincode VARCHAR(10),
    city VARCHAR(100) DEFAULT 'Ahmedabad',
    state VARCHAR(100) DEFAULT 'Gujarat',
    is_24x7 BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Setup Instructions

### Prerequisites
- Google Maps API key with the following APIs enabled:
  - Distance Matrix API
  - Directions API
  - Places API
- Java 11+ with Spring Boot

### Sample Data
Include the pharmacies given in `pharmacies-dataset.json` these data pharmacies in your seed data.
## Evaluation Criteria

### Must Haves (70%)
- Working API endpoints with correct HTTP methods
- Successful integration with all three Google Maps APIs
- Proper error handling and validation
- Clear documentation and setup instructions
- Database with sample pharmacy data

### Good to Have (20%)
- Response caching implementation
- Comprehensive input validation
- Consistent API response format
- Proper logging

### Nice to Have (10%)
- Unit tests for key functions
- Docker containerization
- API documentation (Swagger/OpenAPI)
- Rate limiting implementation

## Submission Guidelines

### Deliverables
1. **Source Code**: Complete project with all endpoints
2. **Video demo**: Short video walkthrough of the API functionality.
3. **README.md**: Setup instructions, API documentation, sample requests
4. **Environment File**: `.env` with required variables
5. **Database**: SQL file or migration scripts for sample data

### README Must Include
- Clear setup and installation steps
- Any assumptions or design decisions made

### Testing Your Submission
Before submitting, ensure:
- All endpoints return valid responses
- Google Maps API integration works correctly
- Sample data is properly seeded
- README instructions are accurate and complete

## Curl Commands for Testing

```bash
# Get distances to all pharmacies from Satellite area
curl "http://localhost:8080/api/pharmacies/distances?lat=23.0225&lng=72.5714&mode=driving"

# Get directions to Apollo Pharmacy Satellite
curl "http://localhost:8080/api/pharmacies/1/directions?fromLat=23.0225&fromLng=72.5714&mode=walking"

# Find nearby pharmacies around Apollo Satellite (2km radius)
curl "http://localhost:8080/api/pharmacies/1/nearby?radius=2000"

# Find nearby pharmacies with chain filter
curl "http://localhost:8080/api/pharmacies/1/nearby?radius=3000&chains=MedPlus,Wellness Forever"

# Search for pharmacy locations in Ahmedabad
curl "http://localhost:8080/api/places/search?query=pharmacy+near+vastrapur+ahmedabad"

# Search for Jan Aushadhi stores specifically
curl "http://localhost:8080/api/places/search?query=Jan+Aushadhi+ahmedabad&types=pharmacy"
```
