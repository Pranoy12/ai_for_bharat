# Implementation Plan: Aushadh-Verify

## Overview

This implementation plan breaks down the Aushadh-Verify system into discrete coding tasks following a microservices architecture. The system will be built incrementally, starting with core verification services, then adding offline capabilities, enforcement reporting, and voice interfaces. Each task builds on previous work, with property-based tests integrated throughout to validate correctness.

## Technology Stack

- **Mobile App**: React Native (TypeScript)
- **API Gateway**: AWS API Gateway with Lambda authorizer
- **Orchestrator**: Node.js/TypeScript on AWS Lambda
- **Visual Verification Agent**: Python 3.11 on AWS Lambda
- **Traceability Agent**: Java 17 with Spring Boot on AWS ECS Fargate
- **Voice Agent**: Python 3.11 on AWS Lambda
- **Enforcement Agent**: Python 3.11 on AWS Lambda
- **Edge AI**: TensorFlow Lite
- **Testing**: Jest + fast-check (TypeScript), Pytest + Hypothesis (Python), JUnit + jqwik (Java)

## Tasks

- [ ] 1. Set up project infrastructure and shared components
  - Create monorepo structure with separate packages for each microservice
  - Set up AWS CDK infrastructure as code for all services
  - Configure CI/CD pipeline with GitHub Actions
  - Set up shared TypeScript types and interfaces package
  - Configure DynamoDB tables for verification logs and user profiles
  - Set up S3 buckets for images and models
  - Configure SQS queues and SNS topics for inter-service communication
  - _Requirements: 13.1, 13.2_

- [ ] 2. Implement API Gateway and authentication
  - [ ] 2.1 Create API Gateway with REST endpoints
    - Define OpenAPI specification for all endpoints
    - Configure CORS and rate limiting
    - Set up request/response validation
    - _Requirements: 10.4_
  
  - [ ] 2.2 Implement AWS Cognito authentication with OTP
    - Configure Cognito user pool with phone number authentication
    - Implement OTP generation and verification flow
    - Create Lambda authorizer for API Gateway
    - _Requirements: 10.1, 10.2_
  
  - [ ]* 2.3 Write property tests for authentication
    - **Property 52: OTP authentication**
    - **Validates: Requirements 10.2**
  
  - [ ]* 2.4 Write unit tests for API endpoints
    - Test request validation and error responses
    - Test authentication flow with valid/invalid OTPs
    - _Requirements: 10.1, 10.2_

- [ ] 3. Implement Verification Orchestrator Service
  - [ ] 3.1 Create orchestrator Lambda function
    - Implement request handler for verification requests
    - Set up SQS message publishing to agent queues
    - Implement result aggregation logic
    - Create classification logic based on visual + blockchain results
    - _Requirements: 3.8, 13.3_
  
  - [ ] 3.2 Implement circuit breaker pattern
    - Create circuit breaker for each external service
    - Configure failure thresholds and timeouts
    - Implement fallback strategies
    - _Requirements: 13.4_
  
  - [ ]* 3.3 Write property test for result aggregation
    - **Property 16: Result aggregation**
    - **Validates: Requirements 3.8**
  
  - [ ]* 3.4 Write property test for fault isolation
    - **Property 67: Fault isolation**
    - **Validates: Requirements 13.4**
  
  - [ ]* 3.5 Write unit tests for orchestration logic
    - Test timeout handling
    - Test partial result scenarios
    - Test circuit breaker state transitions
    - _Requirements: 3.8, 13.4_

- [ ] 4. Checkpoint - Ensure orchestrator tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. Implement Visual Verification Agent Service
  - [ ] 5.1 Create Python Lambda function for visual analysis
    - Implement image validation (format, resolution, quality)
    - Integrate Amazon Rekognition Custom Labels API
    - Implement confidence score calculation
    - Create micro-defect detection logic
    - _Requirements: 1.1, 1.2, 1.3, 1.5, 1.9_
  
  - [ ] 5.2 Implement OCR for Batch ID extraction
    - Integrate Amazon Textract for text extraction
    - Implement pattern matching for Indian pharmaceutical batch formats
    - Create format validation logic
    - _Requirements: 2.1, 2.2, 2.3, 2.5_
  
  - [ ]* 5.3 Write property test for image format validation
    - **Property 1: Image format and resolution validation**
    - **Validates: Requirements 1.1, 1.9**
  
  - [ ]* 5.4 Write property test for confidence score bounds
    - **Property 2: Confidence score bounds**
    - **Validates: Requirements 1.5**
  
  - [ ]* 5.5 Write property test for confidence-based classification
    - **Property 3: Confidence-based classification**
    - **Validates: Requirements 1.6, 1.7, 1.8**
  
  - [ ]* 5.6 Write property test for Batch ID pattern matching
    - **Property 6: Batch ID pattern matching**
    - **Validates: Requirements 2.2, 2.3**
  
  - [ ]* 5.7 Write property test for Batch ID format validation
    - **Property 7: Batch ID format validation**
    - **Validates: Requirements 2.5**
  
  - [ ]* 5.8 Write property test for extraction error handling
    - **Property 8: Batch ID extraction error handling**
    - **Validates: Requirements 2.4**
  
  - [ ]* 5.9 Write unit tests for visual agent
    - Test image quality rejection scenarios
    - Test Rekognition API integration with mocks
    - Test OCR extraction with sample images
    - _Requirements: 1.1, 1.9, 2.1, 2.2_

- [ ] 6. Implement Traceability Agent Service
  - [ ] 6.1 Create Java Spring Boot service for blockchain verification
    - Set up Spring Boot application with ECS deployment config
    - Implement Amazon QLDB query logic
    - Create supply chain validation logic
    - Implement refilled bottle detection (recall + geographic mismatch)
    - Set up Redis caching for frequently queried batches
    - _Requirements: 3.1, 3.2, 3.3, 3.6, 3.7_
  
  - [ ] 6.2 Implement SQS consumer for verification requests
    - Create SQS message listener
    - Implement message processing and error handling
    - Configure retry and dead-letter queue
    - _Requirements: 13.2_
  
  - [ ]* 6.3 Write property test for blockchain query performance
    - **Property 9: Blockchain query performance**
    - **Validates: Requirements 3.1**
  
  - [ ]* 6.4 Write property test for data completeness
    - **Property 10: Blockchain data completeness**
    - **Validates: Requirements 3.2**
  
  - [ ]* 6.5 Write property test for supply chain validation
    - **Property 11: Supply chain validation**
    - **Validates: Requirements 3.3**
  
  - [ ]* 6.6 Write property test for non-existent batch detection
    - **Property 12: Non-existent batch detection**
    - **Validates: Requirements 3.4**
  
  - [ ]* 6.7 Write property test for refilled bottle detection (recall)
    - **Property 14: Refilled bottle detection - recall status**
    - **Validates: Requirements 3.6**
  
  - [ ]* 6.8 Write property test for refilled bottle detection (geographic)
    - **Property 15: Refilled bottle detection - geographic mismatch**
    - **Validates: Requirements 3.7**
  
  - [ ]* 6.9 Write unit tests for traceability agent
    - Test QLDB query with mocked responses
    - Test cache hit/miss scenarios
    - Test supply chain anomaly detection
    - _Requirements: 3.1, 3.3, 3.5_

- [ ] 7. Set up Amazon QLDB blockchain ledger
  - [ ] 7.1 Create QLDB ledger and tables
    - Define Batches and SupplyChain tables
    - Create indexes for fast batch lookup
    - Set up IAM permissions for traceability agent
    - _Requirements: 3.1_
  
  - [ ] 7.2 Populate test data in QLDB
    - Create sample batch records with supply chains
    - Include edge cases (recalls, geographic mismatches)
    - Create broken chain scenarios for testing
    - _Requirements: 3.3, 3.4, 3.5_

- [ ] 8. Checkpoint - Ensure core verification flow works
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Implement Synthetic Data Generation with SageMaker
  - [ ] 9.1 Create SageMaker pipeline for synthetic data generation
    - Implement texture distortion algorithms
    - Implement font kerning error simulation
    - Implement color shift and printing defect simulation
    - Create labeling logic for generated images
    - _Requirements: 8.1, 8.2, 8.3, 8.4_
  
  - [ ]* 9.2 Write property test for synthetic defect variety
    - **Property 40: Synthetic defect variety**
    - **Validates: Requirements 8.2**
  
  - [ ]* 9.3 Write property test for synthetic data labeling
    - **Property 41: Synthetic data labeling**
    - **Validates: Requirements 8.4**
  
  - [ ]* 9.4 Write property test for dataset balance
    - **Property 42: Training dataset balance**
    - **Validates: Requirements 8.5**
  
  - [ ]* 9.5 Write property test for synthetic data realism
    - **Property 44: Synthetic data realism**
    - **Validates: Requirements 8.7**

- [ ] 10. Implement Golden Image Management
  - [ ] 10.1 Create admin API for Golden Image upload
    - Implement upload endpoint with authorization check
    - Create image quality validation logic
    - Implement feature extraction using Rekognition
    - Create versioning logic for multiple images per medicine
    - _Requirements: 7.1, 7.2, 7.3, 7.4_
  
  - [ ]* 10.2 Write property test for upload authorization
    - **Property 36: Golden image upload authorization**
    - **Validates: Requirements 7.1**
  
  - [ ]* 10.3 Write property test for quality validation
    - **Property 37: Golden image quality validation**
    - **Validates: Requirements 7.2**
  
  - [ ]* 10.4 Write property test for feature extraction
    - **Property 38: Golden image feature extraction**
    - **Validates: Requirements 7.3**
  
  - [ ]* 10.5 Write property test for versioning
    - **Property 39: Golden image versioning**
    - **Validates: Requirements 7.4**

- [ ] 11. Train Amazon Rekognition Custom Labels model
  - [ ] 11.1 Prepare training dataset
    - Combine Golden Images with Synthetic Data
    - Split into training/validation sets (80/20)
    - Upload to S3 with proper labeling
    - _Requirements: 8.6_
  
  - [ ] 11.2 Train and deploy Rekognition model
    - Create Rekognition Custom Labels project
    - Train model with prepared dataset
    - Evaluate model performance (target >90% accuracy)
    - Deploy model to inference endpoint
    - _Requirements: 1.2, 8.6_

- [ ] 12. Implement Edge AI Model for offline mode
  - [ ] 12.1 Create TensorFlow Lite model
    - Train MobileNetV3-Small on synthetic data
    - Quantize to INT8 for mobile deployment
    - Validate quantized model accuracy (target >85%)
    - Export to TFLite format
    - _Requirements: 4.1, 4.3, 4.4_
  
  - [ ] 12.2 Implement model update mechanism
    - Create S3 endpoint for model download
    - Implement version checking logic
    - Create model signature validation
    - Implement rollback mechanism
    - _Requirements: 4.9_
  
  - [ ]* 12.3 Write property test for auto-update
    - **Property 22: Edge model auto-update**
    - **Validates: Requirements 4.9**

- [ ] 13. Implement mobile app core functionality
  - [ ] 13.1 Set up React Native project structure
    - Initialize React Native project with TypeScript
    - Set up navigation (React Navigation)
    - Configure state management (Redux Toolkit)
    - Set up API client with authentication
    - _Requirements: 12.1_
  
  - [ ] 13.2 Implement camera module
    - Integrate react-native-camera
    - Create real-time guidance overlay for alignment/lighting
    - Implement image capture and compression
    - _Requirements: 12.2_
  
  - [ ] 13.3 Implement offline mode with TensorFlow Lite
    - Integrate TensorFlow Lite React Native
    - Implement connectivity detection
    - Create automatic mode switching logic
    - Implement local queue for offline verifications
    - Set up SQLite for local storage
    - _Requirements: 4.1, 4.2, 4.3, 4.5_
  
  - [ ] 13.4 Implement verification results UI
    - Create color-coded result display (green/red/yellow/orange)
    - Implement progress indicator
    - Create detailed result view with defects and reasoning
    - _Requirements: 12.3, 12.4_
  
  - [ ]* 13.5 Write property test for offline mode switching
    - **Property 17: Automatic offline mode switching**
    - **Validates: Requirements 4.2, 4.3**
  
  - [ ]* 13.6 Write property test for offline queue
    - **Property 18: Offline verification queue**
    - **Validates: Requirements 4.5**
  
  - [ ]* 13.7 Write property test for sync
    - **Property 19: Offline-to-online synchronization**
    - **Validates: Requirements 4.6**
  
  - [ ]* 13.8 Write property test for offline indicator
    - **Property 20: Offline mode indicator**
    - **Validates: Requirements 4.7**
  
  - [ ]* 13.9 Write property test for color-coded display
    - **Property 63: Color-coded result display**
    - **Validates: Requirements 12.4**

- [ ] 14. Checkpoint - Ensure mobile app core works
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 15. Implement Vernacular Voice Agent Service
  - [ ] 15.1 Create Python Lambda function for voice processing
    - Integrate Amazon Transcribe for Hindi/Marathi
    - Implement custom vocabulary for pharmaceutical terms
    - Create query understanding logic
    - _Requirements: 6.1, 6.2, 6.3_
  
  - [ ] 15.2 Integrate Amazon Bedrock for response generation
    - Configure Claude 3.5 Sonnet model
    - Implement prompt engineering for medical verification context
    - Set up Amazon Bedrock Guardrails
    - Create guardrail configuration (denied topics, content filters)
    - _Requirements: 6.4, 9.7_
  
  - [ ] 15.3 Integrate Amazon Polly for voice synthesis
    - Configure Hindi and Marathi voices (Aditi)
    - Implement language selection logic with Hindi default
    - Create voice warning templates
    - _Requirements: 6.5, 6.6, 6.7, 6.8_
  
  - [ ]* 15.4 Write property test for multi-language support
    - **Property 29: Multi-language voice support**
    - **Validates: Requirements 6.2**
  
  - [ ]* 15.5 Write property test for voice output language
    - **Property 32: Voice output language selection**
    - **Validates: Requirements 6.6, 6.7**
  
  - [ ]* 15.6 Write property test for counterfeit warning
    - **Property 31: Counterfeit voice warning**
    - **Validates: Requirements 6.5**
  
  - [ ]* 15.7 Write property test for voice content completeness
    - **Property 33: Voice content completeness**
    - **Validates: Requirements 6.8**
  
  - [ ]* 15.8 Write property test for out-of-scope queries
    - **Property 50: Out-of-scope query handling**
    - **Validates: Requirements 9.8**
  
  - [ ]* 15.9 Write property test for medical advice prevention
    - **Property 51: Medical advice prevention**
    - **Validates: Requirements 9.9**
  
  - [ ]* 15.10 Write unit tests for voice agent
    - Test Transcribe integration with sample audio
    - Test Bedrock guardrail triggering
    - Test Polly voice generation
    - _Requirements: 6.1, 6.4, 6.5_

- [ ] 16. Implement voice interface in mobile app
  - [ ] 16.1 Add voice recording capability
    - Integrate react-native-audio-recorder
    - Create voice input UI
    - Implement audio upload to API
    - _Requirements: 6.1_
  
  - [ ] 16.2 Add voice playback capability
    - Integrate react-native-sound
    - Implement audio download and playback
    - Create replay button
    - _Requirements: 6.9_
  
  - [ ] 16.3 Implement language preference persistence
    - Create settings screen for language selection
    - Store preference in local storage
    - Apply preference to all voice interactions
    - _Requirements: 6.10_
  
  - [ ]* 16.4 Write property test for voice replay
    - **Property 34: Voice replay availability**
    - **Validates: Requirements 6.9**
  
  - [ ]* 16.5 Write property test for preference persistence
    - **Property 35: Language preference persistence**
    - **Validates: Requirements 6.10**

- [ ] 17. Implement Enforcement Agent Service
  - [ ] 17.1 Create Python Lambda function for enforcement
    - Implement SNS trigger for counterfeit detections
    - Integrate Amazon Location Service for GPS capture
    - Create report data aggregation logic
    - _Requirements: 5.1, 5.2_
  
  - [ ] 17.2 Implement report generation with Bedrock
    - Create prompt template for formal grievance reports
    - Integrate Bedrock for report text generation
    - Implement report formatting for CDSCO
    - _Requirements: 5.4_
  
  - [ ] 17.3 Implement CDSCO API submission
    - Create CDSCO API client with authentication
    - Implement retry logic with exponential backoff
    - Create manual review queue for failed submissions
    - Implement admin notification for failures
    - _Requirements: 5.5, 5.7, 5.8_
  
  - [ ] 17.4 Implement GPS fallback logic
    - Create last known location storage
    - Implement staleness timestamp
    - _Requirements: 5.9_
  
  - [ ]* 17.5 Write property test for automatic triggering
    - **Property 23: Automatic enforcement triggering**
    - **Validates: Requirements 5.1**
  
  - [ ]* 17.6 Write property test for report completeness
    - **Property 24: Enforcement report completeness**
    - **Validates: Requirements 5.3, 9.2**
  
  - [ ]* 17.7 Write property test for CDSCO submission with retry
    - **Property 25: CDSCO submission with retry**
    - **Validates: Requirements 5.5, 5.7**
  
  - [ ]* 17.8 Write property test for failed submission handling
    - **Property 26: Failed submission handling**
    - **Validates: Requirements 5.8**
  
  - [ ]* 17.9 Write property test for GPS fallback
    - **Property 28: GPS fallback**
    - **Validates: Requirements 5.9**
  
  - [ ]* 17.10 Write unit tests for enforcement agent
    - Test report generation with various inputs
    - Test retry backoff timing
    - Test admin notification
    - _Requirements: 5.4, 5.7, 5.8_

- [ ] 18. Checkpoint - Ensure enforcement flow works
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 19. Implement AI Guardrails and Responsible AI features
  - [ ] 19.1 Implement secondary verification logic
    - Create logic to require additional checks for low confidence
    - Implement inconclusive result handling
    - _Requirements: 9.1, 9.6_
  
  - [ ] 19.2 Implement false positive feedback system
    - Create API endpoint for feedback submission
    - Implement feedback logging with images
    - Create feedback storage in DynamoDB
    - _Requirements: 9.3_
  
  - [ ] 19.3 Implement false positive rate monitoring
    - Create CloudWatch metric for false positive rate
    - Implement 30-day rolling window calculation
    - Create alarm for >5% threshold
    - _Requirements: 9.4_
  
  - [ ]* 19.4 Write property test for secondary verification
    - **Property 45: Secondary verification requirement**
    - **Validates: Requirements 9.1**
  
  - [ ]* 19.5 Write property test for false positive logging
    - **Property 46: False positive logging**
    - **Validates: Requirements 9.3**
  
  - [ ]* 19.6 Write property test for FP rate monitoring
    - **Property 47: False positive rate monitoring**
    - **Validates: Requirements 9.4**
  
  - [ ]* 19.7 Write property test for confidence communication
    - **Property 48: Confidence communication**
    - **Validates: Requirements 9.5**
  
  - [ ]* 19.8 Write property test for inconclusive recommendation
    - **Property 49: Inconclusive result recommendation**
    - **Validates: Requirements 9.6**

- [ ] 20. Implement security and privacy features
  - [ ] 20.1 Implement data encryption at rest
    - Enable DynamoDB encryption
    - Enable S3 bucket encryption
    - Implement field-level encryption for PII
    - _Requirements: 10.3_
  
  - [ ] 20.2 Implement TLS 1.3 for all communications
    - Configure API Gateway with TLS 1.3
    - Configure ALB with TLS 1.3
    - Verify all service-to-service calls use TLS
    - _Requirements: 10.4_
  
  - [ ] 20.3 Implement enforcement report access control
    - Create CDSCO user role in Cognito
    - Implement authorization checks for report endpoints
    - Create audit logging for report access
    - _Requirements: 10.5_
  
  - [ ] 20.4 Implement data deletion with anonymization
    - Create data deletion API endpoint
    - Implement PII removal logic
    - Implement anonymization for verification statistics
    - Create 30-day deletion workflow
    - _Requirements: 10.6_
  
  - [ ]* 20.5 Write property test for data encryption
    - **Property 53: Data encryption at rest**
    - **Validates: Requirements 10.3**
  
  - [ ]* 20.6 Write property test for TLS version
    - **Property 54: TLS transport security**
    - **Validates: Requirements 10.4**
  
  - [ ]* 20.7 Write property test for access control
    - **Property 55: Enforcement report access control**
    - **Validates: Requirements 10.5**
  
  - [ ]* 20.8 Write property test for data deletion
    - **Property 56: Data deletion with anonymization**
    - **Validates: Requirements 10.6**

- [ ] 21. Implement monitoring and observability
  - [ ] 21.1 Implement verification logging
    - Create structured logging for all verifications
    - Store logs in DynamoDB with TTL
    - Create CloudWatch log groups for each service
    - _Requirements: 11.1_
  
  - [ ] 21.2 Implement performance monitoring
    - Create CloudWatch metrics for response times
    - Create alarm for >10 second responses
    - Set up X-Ray tracing for distributed tracing
    - _Requirements: 11.2_
  
  - [ ] 21.3 Implement model accuracy monitoring
    - Create metric for Edge AI model accuracy
    - Implement <90% accuracy alarm
    - Create automatic retraining trigger
    - _Requirements: 11.3_
  
  - [ ] 21.4 Implement metrics dashboard
    - Create CloudWatch dashboard for key metrics
    - Track daily verification volume
    - Track success rates and counterfeit detection rates
    - _Requirements: 11.4_
  
  - [ ] 21.5 Implement health check endpoints
    - Create /health endpoint for each microservice
    - Implement dependency health checks
    - Configure ECS/Lambda health checks
    - _Requirements: 13.6_
  
  - [ ]* 21.6 Write property test for verification logging
    - **Property 57: Verification logging**
    - **Validates: Requirements 11.1**
  
  - [ ]* 21.7 Write property test for performance alerting
    - **Property 58: Performance degradation alerting**
    - **Validates: Requirements 11.2**
  
  - [ ]* 21.8 Write property test for accuracy monitoring
    - **Property 59: Model accuracy monitoring**
    - **Validates: Requirements 11.3**
  
  - [ ]* 21.9 Write property test for metrics tracking
    - **Property 60: Metrics tracking**
    - **Validates: Requirements 11.4**
  
  - [ ]* 21.10 Write property test for health monitoring
    - **Property 68: Independent health monitoring**
    - **Validates: Requirements 13.6**

- [ ] 22. Implement remaining mobile app features
  - [ ] 22.1 Implement contextual help system
    - Create help text content in Hindi and Marathi
    - Implement context-aware help display
    - Create help overlay for camera guidance
    - _Requirements: 12.6_
  
  - [ ] 22.2 Implement verification history
    - Create history screen showing past verifications
    - Implement local caching of recent results
    - Create detail view for historical verifications
    - _Requirements: 11.1_
  
  - [ ] 22.3 Polish UI/UX
    - Implement loading states and animations
    - Add error handling with user-friendly messages
    - Implement simple language for all text
    - Create onboarding flow for first-time users
    - _Requirements: 12.1, 12.5_
  
  - [ ]* 22.4 Write property test for capture guidance
    - **Property 61: Real-time capture guidance**
    - **Validates: Requirements 12.2**
  
  - [ ]* 22.5 Write property test for progress indication
    - **Property 62: Progress indication**
    - **Validates: Requirements 12.3**
  
  - [ ]* 22.6 Write property test for contextual help
    - **Property 64: Contextual help availability**
    - **Validates: Requirements 12.6**

- [ ] 23. Integration testing and end-to-end flows
  - [ ]* 23.1 Write integration test for complete online verification
    - Test full flow: image capture → API → agents → result display
    - Verify all services communicate correctly
    - _Requirements: 1.1, 3.8, 12.4_
  
  - [ ]* 23.2 Write integration test for offline verification with sync
    - Test offline capture → local processing → sync when online
    - Verify queue management and sync logic
    - _Requirements: 4.2, 4.5, 4.6_
  
  - [ ]* 23.3 Write integration test for counterfeit detection flow
    - Test counterfeit detection → enforcement report → CDSCO submission
    - Verify end-to-end enforcement workflow
    - _Requirements: 5.1, 5.5, 5.6_
  
  - [ ]* 23.4 Write integration test for voice query flow
    - Test voice input → transcription → Bedrock → Polly → playback
    - Verify voice interface works in both languages
    - _Requirements: 6.1, 6.3, 6.5_

- [ ] 24. Performance testing and optimization
  - [ ]* 24.1 Run load tests on verification endpoints
    - Test 1000 concurrent verifications
    - Verify p95 latency < 5 seconds
    - _Requirements: 1.4, 3.1_
  
  - [ ]* 24.2 Run offline mode performance tests
    - Test Edge AI model inference time
    - Verify p95 latency < 8 seconds
    - _Requirements: 4.8_
  
  - [ ]* 24.3 Optimize slow components
    - Profile and optimize bottlenecks
    - Implement caching where appropriate
    - Tune Lambda memory and timeout settings

- [ ] 25. Security testing and hardening
  - [ ]* 25.1 Run penetration tests on API endpoints
    - Test authentication bypass attempts
    - Test input validation with malicious payloads
    - Test authorization for enforcement reports
    - _Requirements: 10.2, 10.5_
  
  - [ ]* 25.2 Run SAST and DAST scans
    - Use SonarQube for static analysis
    - Use OWASP ZAP for dynamic analysis
    - Fix all critical and high severity issues
  
  - [ ]* 25.3 Verify encryption and TLS configuration
    - Test data encryption at rest
    - Verify TLS 1.3 on all endpoints
    - Test certificate validation
    - _Requirements: 10.3, 10.4_

- [ ] 26. Final checkpoint - Complete system validation
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 27. Documentation and deployment preparation
  - [ ] 27.1 Create API documentation
    - Generate OpenAPI documentation
    - Create developer guide for API usage
    - Document authentication flow
  
  - [ ] 27.2 Create deployment runbook
    - Document infrastructure setup steps
    - Create disaster recovery procedures
    - Document monitoring and alerting setup
  
  - [ ] 27.3 Create user documentation
    - Create mobile app user guide in Hindi and Marathi
    - Create video tutorials for medicine verification
    - Create FAQ document
  
  - [ ] 27.4 Prepare production deployment
    - Create production AWS accounts and resources
    - Set up production monitoring and alerting
    - Configure backup and disaster recovery
    - Create rollback procedures

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each property test should run minimum 100 iterations
- Property tests validate universal correctness across all inputs
- Unit tests validate specific examples and edge cases
- Integration tests validate end-to-end flows across services
- All tests must pass before proceeding to next checkpoint
- Security and privacy are critical - do not skip security tasks
- Performance targets: online <5s, offline <8s, API <200ms
