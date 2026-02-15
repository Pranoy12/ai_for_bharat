# Requirements Document: Aushadh-Verify

## Introduction

Aushadh-Verify is a decentralized, AI-driven counterfeit medicine elimination system designed for rural India. The system addresses the critical challenge of fake pharmaceuticals in regions with varying literacy levels and internet connectivity. Using Amazon Managed Blockchain (Hyperledger Fabric), the system creates a shared, immutable ledger connecting manufacturers, regulators, and pharmacies. AWS AI services provide multilingual voice-based verification, offline capability, and automated blockchain-based authenticity proofs.

## Glossary

- **Aushadh_Verify_System**: The complete decentralized counterfeit medicine elimination platform
- **Blockchain_Network**: Amazon Managed Blockchain (Hyperledger Fabric) consortium connecting stakeholders
- **Vision_Module**: Amazon Rekognition Custom Labels component for analyzing packaging and micro-textures
- **Blockchain_Oracle**: Amazon Bedrock (Claude 3.5 Sonnet) agent that validates scans and writes authenticity proofs to blockchain
- **Translation_Service**: Amazon Translate component for multilingual support
- **Voice_Service**: Combined Amazon Polly (text-to-speech) and Transcribe (speech-to-text) components
- **Edge_Module**: TFLite component for offline local inference
- **Smart_Contract**: Chaincode on Hyperledger Fabric managing batch lifecycle and quality flags
- **Rural_Pharmacist**: Healthcare worker in rural areas who verifies medicine authenticity
- **Pharma_Company**: Pharmaceutical manufacturer registering batches on the blockchain
- **Government_Auditor**: Regulatory authority with read access to all blockchain transactions
- **Consortium_Member**: Authorized participant in the blockchain network (Manufacturer, Regulator, or Pharmacy)
- **Batch_ID**: Unique identifier for a pharmaceutical batch registered on blockchain
- **Authenticity_Proof**: Blockchain record containing scan results and validation from Blockchain_Oracle
- **Verification_Report**: Document containing analysis results and authenticity determination
- **Suspicious_Packaging**: Medicine packaging that exhibits anomalies detected by the system
- **Local_Dialect**: Regional language spoken by the pharmacist (Hindi, Malayalam, Tamil, etc.)
- **Transaction_Hash**: Unique identifier for a blockchain transaction
- **Counterfeit_Flag**: Smart contract event triggered when counterfeit probability exceeds threshold

## Requirements

### Requirement 1: Pharmaceutical Batch Registration

**User Story:** As a pharma company, I want to register new medicine batches on the blockchain, so that the entire supply chain can verify authenticity against the immutable record.

#### Acceptance Criteria

1. WHEN a Pharma_Company creates a new batch, THE Aushadh_Verify_System SHALL register the batch on the Blockchain_Network with a unique Batch_ID
2. THE Smart_Contract SHALL store batch metadata including manufacturer identifier, production date, expiry date, medicine name, and packaging signature
3. WHEN batch registration is complete, THE Blockchain_Network SHALL return a Transaction_Hash as proof of registration
4. THE Aushadh_Verify_System SHALL generate a QR code containing the Batch_ID and Transaction_Hash for physical packaging
5. WHEN a batch is registered, THE Smart_Contract SHALL initialize the batch status as "Registered" and set the current owner to the Pharma_Company
6. THE Blockchain_Network SHALL ensure only authorized Pharma_Company members can register new batches

### Requirement 2: Multilingual Voice Input for Suspicious Activity Reporting

**User Story:** As a non-English speaking rural pharmacist, I want to describe suspicious medicine packaging in my local language using voice, so that I can report concerns without language barriers or literacy requirements.

#### Acceptance Criteria

1. WHERE a pharmacist speaks in a local dialect, WHEN voice input is received, THE Voice_Service SHALL transcribe the speech to text in the source language
2. WHEN transcribed text is available, THE Translation_Service SHALL translate the text to English for processing
3. THE Aushadh_Verify_System SHALL support Hindi, Malayalam, Tamil, Telugu, Bengali, Marathi, Gujarati, and Kannada languages
4. WHEN translation is complete, THE Blockchain_Oracle SHALL process the translated input for verification workflow
5. IF voice input quality is insufficient for transcription, THEN THE Voice_Service SHALL request the pharmacist to repeat the input
6. WHEN verification results are generated, THE Translation_Service SHALL translate the results back to the pharmacist's source language
7. THE Voice_Service SHALL convert translated results to speech in the pharmacist's local dialect
8. WHEN voice description is recorded, THE Blockchain_Oracle SHALL include the transcribed and translated text in the blockchain record

### Requirement 3: Visual Packaging Verification and Blockchain Oracle Validation

**User Story:** As a rural pharmacist, I want to scan medicine packaging with a camera, so that the system can detect visual anomalies and write an authenticity proof to the blockchain.

#### Acceptance Criteria

1. WHEN a pharmacist captures an image of medicine packaging, THE Vision_Module SHALL analyze the image for micro-texture anomalies
2. THE Vision_Module SHALL detect packaging inconsistencies including color variations, font irregularities, and hologram defects
3. WHEN analysis is complete, THE Vision_Module SHALL generate a confidence score between 0 and 100 indicating authenticity likelihood
4. THE Vision_Module SHALL extract visible Batch_ID information from packaging images or QR codes
5. WHEN the Vision_Module completes analysis, THE Blockchain_Oracle SHALL validate the scan result against blockchain-registered batch data
6. THE Blockchain_Oracle SHALL query the Smart_Contract to retrieve the registered packaging signature for the scanned Batch_ID
7. WHEN validation is complete, THE Blockchain_Oracle SHALL write an Authenticity_Proof to the Blockchain_Network containing scan metadata, confidence score, timestamp, and pharmacist identifier
8. THE Authenticity_Proof SHALL include the Transaction_Hash linking the scan to the original batch registration
9. WHEN the confidence score is greater than or equal to 90, THE Blockchain_Oracle SHALL mark the Authenticity_Proof as "Verified_Authentic"
10. WHEN multiple images of the same package are provided, THE Vision_Module SHALL perform comparative analysis across all images before sending results to the Blockchain_Oracle

### Requirement 4: Automated Counterfeit Flagging via Smart Contract

**User Story:** As a government auditor, I want the system to automatically flag suspicious batches across the entire network when counterfeits are detected, so that all stakeholders are immediately alerted.

#### Acceptance Criteria

1. WHEN a scan indicates a confidence score below 10 (greater than 90% probability of counterfeit), THE Blockchain_Oracle SHALL trigger a Counterfeit_Flag event in the Smart_Contract
2. THE Smart_Contract SHALL update the batch status to "Flagged_Counterfeit" and broadcast the event to all Consortium_Members
3. WHEN a Counterfeit_Flag event is triggered, THE Smart_Contract SHALL prevent any further transfers of the flagged batch
4. THE Aushadh_Verify_System SHALL notify the Government_Auditor and the registered Pharma_Company of the counterfeit detection within 60 seconds
5. WHEN a batch is flagged, THE Smart_Contract SHALL record the flagging pharmacist's location and identifier for investigation
6. THE Blockchain_Network SHALL maintain an immutable history of all Counterfeit_Flag events for regulatory review

### Requirement 5: Offline Verification Capability

**User Story:** As a rural pharmacist in an area with unreliable internet, I want to verify medicines offline, so that connectivity issues do not prevent me from detecting counterfeits.

#### Acceptance Criteria

1. WHERE internet connectivity is unavailable, WHEN a verification request is initiated, THE Edge_Module SHALL perform local inference using cached models
2. THE Edge_Module SHALL store verification results locally until connectivity is restored
3. WHEN connectivity is restored, THE Edge_Module SHALL synchronize all pending verification results with the Blockchain_Oracle for blockchain recording
4. THE Edge_Module SHALL maintain a local cache of at least 10,000 recently registered Batch_IDs with their packaging signatures for offline lookup
5. WHILE operating offline, THE Aushadh_Verify_System SHALL provide visual indicators showing offline mode status
6. THE Edge_Module SHALL update local models and batch cache when connectivity is available
7. WHEN operating offline, THE Edge_Module SHALL generate preliminary Authenticity_Proofs that are finalized on the blockchain once connectivity returns

### Requirement 6: Blockchain-Based Batch Traceability

**User Story:** As a government auditor, I want an immutable blockchain record of all medicine batch movements and verifications, so that I can track counterfeit patterns and ensure data integrity for legal proceedings.

#### Acceptance Criteria

1. WHEN a verification is performed, THE Blockchain_Oracle SHALL create an immutable Authenticity_Proof on the Blockchain_Network containing timestamp, Batch_ID, location, confidence score, and pharmacist identifier
2. THE Blockchain_Network SHALL maintain cryptographic proof of data integrity for all stored records using Hyperledger Fabric's endorsement policies
3. WHEN a Batch_ID is queried, THE Smart_Contract SHALL return the complete verification history and transfer chain for that batch
4. THE Aushadh_Verify_System SHALL prevent modification or deletion of any record once written to the Blockchain_Network
5. WHEN multiple verifications exist for the same Batch_ID, THE Blockchain_Network SHALL preserve all verification attempts with their respective timestamps and Transaction_Hash values
6. THE Smart_Contract SHALL support queries filtering by date range, location, pharmacist, verification result, and consortium member

### Requirement 7: Multilingual Cross-Entity Communication

**User Story:** As a government auditor, I want to communicate with pharmacists and manufacturers in their preferred languages, so that regulatory actions are clearly understood across linguistic boundaries.

#### Acceptance Criteria

1. WHEN a Verification_Report is generated, THE Aushadh_Verify_System SHALL store the report in English as the primary language on the blockchain
2. WHERE a Government_Auditor requests a report in a specific language, THE Translation_Service SHALL translate the report to the requested language
3. THE Aushadh_Verify_System SHALL maintain multilingual metadata in Authenticity_Proofs preserving original voice input language alongside English translations
4. WHEN generating reports, THE Aushadh_Verify_System SHALL include both the original pharmacist's voice description and its English translation
5. THE Blockchain_Network SHALL store language metadata for each verification record indicating the source language used
6. WHEN a Pharma_Company or Government_Auditor sends notifications, THE Translation_Service SHALL translate messages to the recipient's preferred language

### Requirement 8: Smart Contract Batch Transfer Logic

**User Story:** As a pharma company, I want to transfer batch ownership through the supply chain on the blockchain, so that each custody change is recorded immutably.

#### Acceptance Criteria

1. WHEN a Consortium_Member initiates a batch transfer, THE Smart_Contract SHALL verify the member has current ownership of the batch
2. THE Smart_Contract SHALL update the batch owner to the recipient and record the transfer timestamp and Transaction_Hash
3. WHEN a batch is flagged as counterfeit, THE Smart_Contract SHALL reject any transfer attempts for that batch
4. THE Smart_Contract SHALL maintain a complete chain of custody showing all ownership transfers from manufacturer to end pharmacy
5. WHEN a transfer is complete, THE Smart_Contract SHALL emit a transfer event visible to all Consortium_Members
6. THE Aushadh_Verify_System SHALL require digital signatures from both sender and receiver to authorize transfers

### Requirement 9: Blockchain Oracle Orchestration

**User Story:** As a rural pharmacist, I want the system to automatically coordinate scanning, blockchain validation, and reporting, so that I can complete verifications with minimal technical knowledge.

#### Acceptance Criteria

1. WHEN a verification is initiated, THE Blockchain_Oracle SHALL coordinate the Vision_Module, Smart_Contract queries, and Translation_Service in sequence
2. THE Blockchain_Oracle SHALL query the Smart_Contract for historical verification data and transfer history on the detected Batch_ID
3. WHEN historical data indicates previous counterfeit flags for a Batch_ID, THE Blockchain_Oracle SHALL elevate the alert priority
4. THE Blockchain_Oracle SHALL generate a comprehensive Verification_Report combining visual analysis, blockchain history, and confidence assessment
5. WHEN verification is complete, THE Blockchain_Oracle SHALL write the Authenticity_Proof to the Blockchain_Network and notify relevant Government_Auditors if counterfeit is detected
6. IF any component fails during orchestration, THEN THE Blockchain_Oracle SHALL log the failure and provide a partial report with available data
7. THE Blockchain_Oracle SHALL validate that the scanned Batch_ID exists on the blockchain before performing detailed analysis

### Requirement 10: Synthetic Training Data Generation

**User Story:** As a system administrator, I want to generate synthetic counterfeit packaging data, so that the Vision_Module can be trained to detect fake medicines without requiring large quantities of actual counterfeits.

#### Acceptance Criteria

1. THE Aushadh_Verify_System SHALL use Amazon SageMaker to generate synthetic images of counterfeit drug packaging
2. WHEN generating synthetic data, THE Aushadh_Verify_System SHALL introduce realistic anomalies including blurred text, color shifts, and texture variations
3. THE Aushadh_Verify_System SHALL maintain a training dataset with at least 80% synthetic data and 20% real counterfeit samples
4. WHEN new legitimate packaging designs are registered on the blockchain, THE Aushadh_Verify_System SHALL automatically generate corresponding synthetic counterfeit variations
5. THE Aushadh_Verify_System SHALL retrain the Vision_Module monthly using updated synthetic datasets
6. THE Aushadh_Verify_System SHALL link Rekognition metadata (confidence scores, detected features) to the Transaction_Hash of the corresponding blockchain record

### Requirement 11: Government Auditor Dashboard and Analytics

**User Story:** As a government auditor, I want to view aggregated counterfeit detection statistics from the blockchain, so that I can identify geographic patterns and prioritize enforcement actions.

#### Acceptance Criteria

1. THE Aushadh_Verify_System SHALL provide a dashboard displaying counterfeit detection rates by region, time period, and medicine type
2. WHEN a Government_Auditor queries the dashboard, THE Smart_Contract SHALL aggregate verification data while maintaining individual record immutability
3. THE Aushadh_Verify_System SHALL generate heat maps showing geographic concentration of counterfeit detections
4. WHEN suspicious patterns are detected across multiple locations for the same Batch_ID, THE Aushadh_Verify_System SHALL automatically alert Government_Auditors
5. THE Aushadh_Verify_System SHALL support export of aggregated data in CSV and PDF formats for regulatory reporting
6. THE Aushadh_Verify_System SHALL display the complete supply chain path for any flagged batch showing all custody transfers

### Requirement 12: Consortium Member Authentication and Authorization

**User Story:** As a government auditor, I want to ensure only authorized consortium members can perform their respective operations, so that the blockchain maintains data integrity and prevents misuse.

#### Acceptance Criteria

1. WHEN a Consortium_Member attempts to interact with the blockchain, THE Aushadh_Verify_System SHALL require authentication using digital certificates issued by the consortium
2. THE Blockchain_Network SHALL maintain a registry of authorized members with their roles (Pharma_Company, Rural_Pharmacist, Government_Auditor) and permissions
3. WHEN a transaction is submitted, THE Smart_Contract SHALL verify the member's role has permission to perform the requested operation
4. IF an unauthorized user attempts blockchain interaction, THEN THE Blockchain_Network SHALL reject the transaction and log the attempt
5. THE Aushadh_Verify_System SHALL support member credential revocation by Government_Auditors
6. THE Smart_Contract SHALL enforce role-based access control where Pharma_Companies can register batches, Rural_Pharmacists can verify batches, and Government_Auditors have read-only access to all data
