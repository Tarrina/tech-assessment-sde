# InCredit: Credit Risk Assessment API

## Overview
Build a REST API service for InCredit India's credit risk assessment system. This project focuses on developing a statistical analysis-powered API that evaluates credit repayment likelihood for potential borrowers based on their historical financial behavior using simple regression analysis.

## Time Estimate
**3 hours**

## Business Context
InCredit India is a growing digital lending company providing small to medium-sized personal and purchase financing credits across India. The company needs a reliable credit risk assessment system to minimize defaults, manage portfolio effectively, and offer competitive terms to creditworthy individuals.

## Tech Stack Requirements
- **Language**: Java (Spring Boot)
- **Analytics**: Built-in statistical calculations using Java libraries (Apache Commons Math)
- **Data Format**: JSON processing for customer order history
- **Documentation**: README with setup instructions

## Project Requirements

### Core Features

#### 1. Credit Risk Assessment
**Endpoint**: `POST /api/assessments`

**Request Body**:
```json
{
  "customerId": "7",
  "requestedCreditAmount": 50000,
  "tenureMonths": 12,
  "purpose": "personal"
}
```

**Functionality**:
- Assess likelihood of on-time repayment using linear regression
- Calculate risk score (0-100, where 100 is highest risk)
- Provide risk category: LOW, MEDIUM, HIGH
- Include confidence level and key statistical factors
- Log all assessment requests for audit

#### 2. Statistical Analysis Service
**Endpoint**: `GET /api/customers/{id}/statistics`

**Functionality**:
- Calculate basic statistical metrics from customer's historical data:
  - Payment history summary (on-time percentage, average payment delay)
  - Credit utilization averages
  - Spending patterns and trends
  - Simple correlation analysis between income and payment behavior
  - Basic regression coefficients for payment prediction
- Return computed statistics in structured format
- Cache statistical calculations for performance

### API Response Examples

#### Credit Risk Assessment Response
```json
{
  "customerId": "7",
  "assessmentId": "asmt_20231201_12345",
  "riskAssessment": {
    "willPayOnTime": true,
    "riskScore": 25,
    "riskCategory": "LOW",
    "confidence": 0.87
  },
  "statisticalFactors": [
    {
      "factor": "payment_history_percentage",    {
      "factor": "payment_history_percentage",
      "value": 100.0,
      "impact": "positive",
      "description": "100% on-time payment rate with excellent track record"
    },
    {
      "factor": "income_to_credit_ratio",
      "value": 2.3,
      "impact": "positive", 
      "description": "Strong income-to-credit ratio of 2.3x"
    }
  ],
  "regressionCoefficients": {
    "paymentHistoryWeight": 0.45,
    "incomeRatioWeight": 0.32,
    "creditUtilizationWeight": -0.18
  },
  "recommendation": "APPROVE",
  "alternativeTerms": {
    "maxRecommendedAmount": 75000,
    "recommendedTenure": 12
  },
  "timestamp": "2023-12-01T10:30:00Z"
}
```

#### Customer Statistics Response
```json
{
  "customerId": "7",
  "statistics": {
    "paymentHistory": {
      "totalOrders": 3,
      "onTimePayments": 3,
      "latePayments": 0,
      "defaults": 0,
      "onTimePercentage": 100.0,
      "avgDaysLate": 2.7,
      "paymentVariance": 4.2
    },    "creditUtilization": {
      "avgCreditAmount": 58333,
      "maxCreditAmount": 75000,
      "totalCreditExtended": 175000,
      "avgUtilization": 82.5,
      "utilizationTrend": "stable"
    },
    "behaviorMetrics": {
      "daysSinceLastOrder": 85,
      "daysSinceRegistration": 1578,
      "monthlyOrderFrequency": 0.57,
      "creditGrowthRate": 0.25
    },    "correlationAnalysis": {
      "incomeVsPaymentCorrelation": 0.45,
      "ageVsRiskCorrelation": -0.12,
      "utilizationVsDefaultCorrelation": 0.28
    }
  },
  "computedAt": "2023-12-01T10:25:00Z"
}
```

## Technical Requirements

### Data Processing
- Parse complex JSON structure with nested order arrays
- Handle date parsing and timezone considerations  
- Validate payment status enums and business rules
- Calculate derived fields (days outstanding, payment ratios)
- Handle missing or inconsistent data gracefully

**Data Validation Rules**:
- `last_payment_date` can be null for DEFAULT status orders
- Payment status must align with `amount_paid_inr` values:
  - PAID: amount_paid_inr = credit_extended_amount_inr
  - PARTIALLY_PAID: 0 < amount_paid_inr < credit_extended_amount_inr  
  - UNPAID: amount_paid_inr = 0
  - OVERDUE: any payment amount with days_outstanding_or_late > 0
  - DEFAULT: typically amount_paid_inr < 50% of credit_extended_amount_inr

### 2. Statistical Analysis Integration
- Implement linear regression for risk assessment
- Support real-time statistical calculations  
- Handle statistical model updates with new data
- Implement basic data normalization and scaling
- Cache statistical calculations for performance

**Statistical Limitations with Current Dataset**:
- Small sample size (10 customers) limits correlation reliability
- Confidence intervals should account for limited data points  
- Risk scoring should include uncertainty bounds
- Regression models may have high variance with limited training data

## Setup Instructions

### Sample Data
Include the customer data given in `payment-history.json` in your seed data.

**Data Quality Notes**:
- Some orders have null `last_payment_date` (handled for DEFAULT status)
- Payment statuses have been validated for consistency
- Days outstanding capped at reasonable business limits
- All monetary values in INR

### Sample Data Structure
The `payment-history.json` file contains customer records with this structure:
```json
[
  {
    "user_id": 1,
    "registration_date": "2022-01-10",
    "city": "Mumbai",
    "state": "Maharashtra",
    "age": 35,
    "approx_annual_income_inr": 2000000,
    "employment_status": "Salaried",
    "orders": [
      {
        "order_id": "THPL-001",
        "order_date": "2022-03-15",
        "order_amount_inr": 50000.00,
        "credit_extended_amount_inr": 50000.00,
        "credit_tenure_months": 3,
        "payment_due_date": "2022-06-15",
        "payment_status": "PAID",
        "last_payment_date": "2022-06-10",
        "amount_paid_inr": 50000.00,
        "days_outstanding_or_late": 0
      },
      {
        "order_id": "THPL-002",
        "order_date": "2023-02-20",
        "order_amount_inr": 30000.00,
        "credit_extended_amount_inr": 25000.00,
        "credit_tenure_months": 6,
        "payment_due_date": "2023-08-20",
        "payment_status": "PAID",
        "last_payment_date": "2023-08-30",
        "amount_paid_inr": 25000.00,
        "days_outstanding_or_late": 10
      }
    ],
    "target_will_pay_on_time_next_credit": true
  }
]
```

## Statistical Analysis Requirements

### Key Metrics Calculation
Implement these statistical calculations for credit risk assessment:

1. **Payment Behavior Statistics**:
   - On-time payment percentage
   - Average payment delay in days
   - Payment amount variance
   - Payment consistency score

2. **Credit Utilization Analysis**:
   - Average credit amount
   - Credit-to-income ratio
   - Credit utilization trend (linear regression)
   - Maximum credit exposure

3. **Financial Pattern Analysis**:
   - Days since registration
   - Order frequency (orders per month)
   - Seasonal payment patterns (if applicable)
   - Income stability indicators

4. **Risk Scoring Algorithm**:
   - Linear regression model for payment prediction
   - Weighted scoring based on historical patterns
   - Risk categorization (LOW/MEDIUM/HIGH)
   - Simple correlation analysis

### Statistical Implementation
- Use linear regression for payment likelihood prediction
- Calculate correlation coefficients between key variables
- Implement basic normalization for fair comparison
- Provide statistical confidence intervals
- Support simple trend analysis

## Evaluation Criteria

### Must Haves (70%)
- Working API endpoints with correct HTTP methods
- Successful JSON data parsing and storage
- Statistical analysis implementation for risk assessment
- Linear regression model for payment prediction

### Good to Have (20%)
- Assessment performance tracking
- Comprehensive error handling and validation
- API documentation (Swagger/OpenAPI)
- Caching and performance optimization

### Nice to Have (10%)
- Statistical confidence intervals and correlation analysis
- Unit and integration tests

## Submission Guidelines

### Deliverables
1. **Source Code**: Complete Spring Boot project with all endpoints
2. **Video demo**: Short video walkthrough of the API functionality
3. **README.md**: Setup instructions, API documentation, sample requests
4. **Database Scripts**: SQL files for schema and sample data
5. **Analysis Report**: Brief explanation of statistical approach and calculations

### README Must Include
- API endpoint documentation with examples
- Database setup instructions
- Statistical analysis implementation details
- Key design decisions and assumptions

### Testing Your Submission
Before submitting, ensure:
- All API endpoints return valid responses
- Data import from JSON works correctly
- Statistical risk assessments are generating reasonable results
- Database constraints and validations work
- Performance is acceptable for 1000+ customers

## Curl Commands for Testing

```bash
# Import customer data
curl -X POST "http://localhost:8080/api/customers/import" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@payment-history.json"

# Get customer details
curl "http://localhost:8080/api/customers/12345"

# Get customer statistics
curl "http://localhost:8080/api/customers/12345/statistics"

# Assess credit risk
curl -X POST "http://localhost:8080/api/assessments" \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "12345",
    "requestedCreditAmount": 50000,
    "tenureMonths": 12,
    "purpose": "personal"
  }'

# Get customer assessment history
curl "http://localhost:8080/api/customers/12345/assessments"

# Get risk analytics
curl "http://localhost:8080/api/analytics/risk-distribution?timeframe=6months"

# Get correlation analysis
curl "http://localhost:8080/api/analytics/correlation-analysis"
```
