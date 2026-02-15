# Design Document: Aushadh-Verify

## Overview

Aushadh-Verify is a decentralized counterfeit medicine detection system that leverages AWS services and blockchain technology to create an immutable, transparent supply chain for pharmaceuticals in rural India. The system addresses three critical challenges:

1. **Language Barriers**: Multilingual voice-based interfaces allow non-English speaking pharmacists to interact naturally
2. **Connectivity Constraints**: Offline-first architecture ensures verification works in areas with unreliable internet
3. **Trust and Traceability**: Blockchain-based immutable records provide cryptographic proof of authenticity

The architecture combines Amazon Managed Blockchain (Hyperledger Fabric) for decentralized consensus, Amazon Bedrock (Claude 3.5 Sonnet) as an intelligent blockchain oracle, Amazon Rekognition for visual analysis, and Amazon Transcribe/Polly/Translate for multilingual support.

## Architecture

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Consortium Members                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   Pharma     │  │    Rural     │  │  Government  │             │
│  │  Companies   │  │  Pharmacies  │  │   Auditors   │             │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘             │
│         │                  │                  │                      │
└─────────┼──────────────────┼──────────────────┼──────────────────────┘
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│              Amazon Managed Blockchain (Hyperledger Fabric)         │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    Smart Contract (Chaincode)                  │ │
│  │  • Batch Registration    • Transfer Logic                     │ │
│  │  • Authenticity Proofs   • Counterfeit Flagging               │ │
│  │  • Access Control        • Query Functions                    │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  Amazon Bedrock (Blockchain Oracle)                 │
│                      Claude 3.5 Sonnet Agent                        │
│  • Validates scan results                                           │
│  • Queries blockchain for batch history                             │
│  • Writes authenticity proofs                                       │
│  • Orchestrates multilingual pipeline                               │
│  • Triggers smart contract events                                   │
└─────────────────────────────────────────────────────────────────────┘
          │                  │                  │
          ▼                  ▼                  ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  Amazon          │ │  Multilingual    │ │  Amazon          │
│  Rekognition     │ │  Services        │ │  SageMaker       │
│  Custom Labels   │ │                  │ │                  │
│                  │ │  • Transcribe    │ │  • Synthetic     │
│  • Micro-texture │ │  • Translate     │ │    Data Gen      │
│  • Packaging     │ │  • Polly         │ │  • Model         │
│    Analysis      │ │                  │ │    Training      │
└──────────────────┘ └──────────────────┘ └──────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         Edge Layer (Rural Devices)                   │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  TFLite Offline Inference                                      │ │
│  │  • Cached models                                               │ │
│  │  • Local batch database (10,000+ entries)                     │ │
│  │  • Sync queue for pending verifications                       │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### Consortium Architecture

The blockchain network operates as a permissioned consortium with three member types:

1. **Pharma Companies (Manufacturers)**
   - Register new batches on blockchain
   - Upload packaging signatures
   - Receive counterfeit alerts
   - Transfer batch ownership

2. **Rural Pharmacies (Verifiers)**
   - Scan and verify medicine packaging
   - Submit voice descriptions
   - Query batch history
   - Cannot modify batch data

3. **Government Auditors (Regulators)**
   - Read-only access to all transactions
   - Query aggregated statistics
   - Revoke member credentials
   - Export compliance reports

Each member authenticates using X.509 certificates issued by the consortium's Certificate Authority (CA). Hyperledger Fabric's Membership Service Provider (MSP) enforces role-based access control.

### Data Flow

**Batch Registration Flow:**
```
Pharma Company → API Gateway → Lambda → Smart Contract → Blockchain
                                    ↓
                              QR Code Generator → Packaging
```

**Verification Flow (Online):**
```
Pharmacist → Mobile App → Camera/Microphone
                ↓
         Vision Module (Rekognition) + Voice Service (Transcribe)
                ↓
         Blockchain Oracle (Bedrock)
                ↓
         Smart Contract Query (batch history)
                ↓
         Write Authenticity Proof → Blockchain
                ↓
         Translation Service → Voice Response (Polly)
```

**Verification Flow (Offline):**
```
Pharmacist → Mobile App → Camera/Microphone
                ↓
         Edge Module (TFLite) + Local Cache
                ↓
         Local Storage (pending queue)
                ↓
    [Connectivity Restored]
                ↓
         Blockchain Oracle → Sync to Blockchain
```

## Components and Interfaces

### 1. Blockchain Oracle (Amazon Bedrock Agent)

The Blockchain Oracle is the intelligent coordinator that bridges off-chain AI services with on-chain smart contracts.

**Responsibilities:**
- Validate scan results from Vision Module
- Query Smart Contract for batch history
- Write Authenticity Proofs to blockchain
- Orchestrate multilingual pipeline
- Trigger counterfeit flag events
- Handle offline sync operations

**Interface:**

```typescript
interface BlockchainOracle {
  // Main verification workflow
  verifyMedicine(request: VerificationRequest): Promise<VerificationResult>
  
  // Blockchain interactions
  queryBatchHistory(batchId: string): Promise<BatchHistory>
  writeAuthenticityProof(proof: AuthenticityProof): Promise<TransactionHash>
  triggerCounterfeitFlag(batchId: string, evidence: Evidence): Promise<void>
  
  // Offline sync
  syncPendingVerifications(pendingQueue: VerificationRequest[]): Promise<SyncResult>
  
  // Orchestration
  orchestrateVerification(
    visionResult: VisionAnalysis,
    voiceInput: TranslatedVoiceInput,
    batchHistory: BatchHistory
  ): Promise<VerificationReport>
}

interface VerificationRequest {
  batchId: string
  images: ImageData[]
  voiceDescription?: VoiceInput
  pharmacistId: string
  location: GeoLocation
  timestamp: number
  isOffline: boolean
}

interface VerificationResult {
  batchId: string
  confidenceScore: number  // 0-100
  status: 'Verified_Authentic' | 'Suspicious' | 'Flagged_Counterfeit'
  authenticityProof: AuthenticityProof
  verificationReport: VerificationReport
  transactionHash: string
  voiceResponse: AudioData  // Translated and synthesized
}

interface BatchHistory {
  batchId: string
  registrationDate: number
  manufacturer: string
  currentOwner: string
  transferChain: Transfer[]
  previousVerifications: AuthenticityProof[]
  counterfeitFlags: CounterfeitFlag[]
  status: 'Registered' | 'In_Transit' | 'Delivered' | 'Flagged_Counterfeit'
}
```

### 2. Vision Module (Amazon Rekognition Custom Labels)

Analyzes medicine packaging images for visual anomalies and authenticity markers.

**Responsibilities:**
- Detect micro-texture anomalies
- Identify packaging inconsistencies (color, font, hologram)
- Extract Batch ID from images/QR codes
- Generate confidence scores
- Perform comparative analysis across multiple images

**Interface:**

```typescript
interface VisionModule {
  analyzePackaging(images: ImageData[]): Promise<VisionAnalysis>
  extractBatchId(image: ImageData): Promise<string | null>
  compareMultipleImages(images: ImageData[]): Promise<ComparativeAnalysis>
  detectAnomalies(image: ImageData): Promise<Anomaly[]>
}

interface VisionAnalysis {
  confidenceScore: number  // 0-100
  anomalies: Anomaly[]
  extractedBatchId: string | null
  packagingFeatures: PackagingFeatures
  metadata: {
    analysisTimestamp: number
    modelVersion: string
    imageQuality: number
  }
}

interface Anomaly {
  type: 'color_variation' | 'font_irregularity' | 'hologram_defect' | 
        'texture_mismatch' | 'blur' | 'missing_element'
  severity: number  // 0-100
  location: BoundingBox
  description: string
}

interface PackagingFeatures {
  colorProfile: ColorHistogram
  textFeatures: TextFeature[]
  hologramPresent: boolean
  microTextureSignature: number[]  // Feature vector
}
```

### 3. Multilingual Services

Handles voice input/output and translation across 8 Indian languages.

**Responsibilities:**
- Transcribe voice input to text (Amazon Transcribe)
- Translate between languages (Amazon Translate)
- Synthesize speech output (Amazon Polly)
- Maintain language metadata in blockchain records

**Interface:**

```typescript
interface MultilingualService {
  // Voice to text
  transcribeVoice(audio: AudioData, sourceLanguage: Language): Promise<TranscriptionResult>
  
  // Translation
  translate(text: string, sourceLanguage: Language, targetLanguage: Language): Promise<string>
  
  // Text to voice
  synthesizeSpeech(text: string, targetLanguage: Language): Promise<AudioData>
  
  // Full pipeline
  processVoiceInput(audio: AudioData, sourceLanguage: Language): Promise<TranslatedVoiceInput>
  generateVoiceResponse(report: VerificationReport, targetLanguage: Language): Promise<AudioData>
}

interface TranscriptionResult {
  text: string
  confidence: number
  language: Language
  timestamp: number
}

interface TranslatedVoiceInput {
  originalText: string
  originalLanguage: Language
  translatedText: string  // English
  confidence: number
}

type Language = 'hi' | 'ml' | 'ta' | 'te' | 'bn' | 'mr' | 'gu' | 'kn' | 'en'
```

### 4. Smart Contract (Hyperledger Fabric Chaincode)

Manages batch lifecycle, authenticity proofs, and access control on the blockchain.

**Responsibilities:**
- Register new batches
- Store and query authenticity proofs
- Manage batch transfers
- Trigger counterfeit flags
- Enforce role-based access control
- Maintain immutable audit trail

**Interface:**

```go
// Smart Contract Functions (Chaincode)

// Batch Management
func RegisterBatch(
  batchId string,
  manufacturer string,
  productionDate int64,
  expiryDate int64,
  medicineName string,
  packagingSignature []byte,
) (string, error)  // Returns transaction hash

func TransferBatch(
  batchId string,
  fromOwner string,
  toOwner string,
  senderSignature []byte,
  receiverSignature []byte,
) (string, error)

func QueryBatch(batchId string) (Batch, error)

// Authenticity Proofs
func WriteAuthenticityProof(
  batchId string,
  pharmacistId string,
  location GeoLocation,
  confidenceScore int,
  scanMetadata ScanMetadata,
  timestamp int64,
) (string, error)  // Returns transaction hash

func QueryAuthenticityProofs(batchId string) ([]AuthenticityProof, error)

// Counterfeit Flagging
func FlagCounterfeit(
  batchId string,
  pharmacistId string,
  location GeoLocation,
  evidence Evidence,
) error

func QueryCounterfeitFlags(filters QueryFilters) ([]CounterfeitFlag, error)

// Access Control
func AuthenticateMember(certificate []byte) (Member, error)
func RevokeMember(memberId string, auditorSignature []byte) error
func QueryMemberPermissions(memberId string) (Permissions, error)

// Analytics
func AggregateVerifications(
  startDate int64,
  endDate int64,
  filters QueryFilters,
) (AggregatedStats, error)
```

### 5. Edge Module (TFLite Offline Inference)

Provides offline verification capability using cached models and batch data.

**Responsibilities:**
- Perform local inference when offline
- Cache recent batch data (10,000+ entries)
- Queue pending verifications for sync
- Update models when connectivity available
- Indicate offline mode status

**Interface:**

```typescript
interface EdgeModule {
  // Offline inference
  verifyOffline(request: VerificationRequest): Promise<PreliminaryResult>
  
  // Cache management
  updateBatchCache(batches: CachedBatch[]): Promise<void>
  queryLocalBatch(batchId: string): Promise<CachedBatch | null>
  getCacheStatus(): CacheStatus
  
  // Sync management
  queueVerification(result: PreliminaryResult): Promise<void>
  getPendingQueue(): PreliminaryResult[]
  syncWithBlockchain(oracle: BlockchainOracle): Promise<SyncResult>
  
  // Model management
  updateModels(modelData: ModelData): Promise<void>
  getModelVersion(): string
}

interface CachedBatch {
  batchId: string
  manufacturer: string
  packagingSignature: number[]  // Feature vector
  expiryDate: number
  cachedAt: number
}

interface PreliminaryResult {
  batchId: string
  confidenceScore: number
  status: 'Preliminary_Authentic' | 'Preliminary_Suspicious'
  timestamp: number
  requiresBlockchainSync: boolean
  localAnalysis: VisionAnalysis
}

interface SyncResult {
  syncedCount: number
  failedCount: number
  errors: SyncError[]
}
```

### 6. Dashboard Service (Government Auditor Interface)

Provides analytics and reporting for regulatory oversight.

**Responsibilities:**
- Aggregate verification statistics
- Generate geographic heat maps
- Export compliance reports
- Display supply chain paths
- Alert on suspicious patterns

**Interface:**

```typescript
interface DashboardService {
  // Analytics
  getCounterfeitStats(filters: AnalyticsFilters): Promise<CounterfeitStats>
  getGeographicHeatMap(filters: AnalyticsFilters): Promise<HeatMapData>
  getSupplyChainPath(batchId: string): Promise<SupplyChainPath>
  
  // Reporting
  exportReport(format: 'CSV' | 'PDF', filters: AnalyticsFilters): Promise<ReportData>
  
  // Alerts
  getAutomaticAlerts(): Promise<Alert[]>
  subscribeToAlerts(callback: (alert: Alert) => void): void
}

interface CounterfeitStats {
  totalVerifications: number
  counterfeitDetections: number
  detectionRate: number
  byRegion: Map<string, RegionStats>
  byMedicineType: Map<string, MedicineStats>
  byTimePeriod: TimeSeriesData[]
}

interface HeatMapData {
  regions: GeoRegion[]
  intensityLevels: Map<string, number>  // Region ID -> counterfeit count
}

interface SupplyChainPath {
  batchId: string
  manufacturer: string
  transfers: Transfer[]
  verifications: AuthenticityProof[]
  currentStatus: string
}
```

### 7. Synthetic Data Generator (Amazon SageMaker)

Generates synthetic counterfeit packaging images for training the Vision Module.

**Responsibilities:**
- Generate synthetic counterfeit variations
- Introduce realistic anomalies
- Maintain 80/20 synthetic/real ratio
- Auto-generate counterfeits for new packaging designs
- Trigger monthly model retraining

**Interface:**

```typescript
interface SyntheticDataGenerator {
  generateCounterfeitVariations(
    legitimatePackaging: ImageData,
    count: number
  ): Promise<SyntheticImage[]>
  
  introduceAnomalies(
    image: ImageData,
    anomalyTypes: AnomalyType[]
  ): Promise<SyntheticImage>
  
  getTrainingDataset(): Promise<TrainingDataset>
  
  triggerRetraining(): Promise<RetrainingJob>
}

interface SyntheticImage {
  imageData: ImageData
  appliedAnomalies: Anomaly[]
  groundTruth: 'counterfeit' | 'authentic'
  metadata: {
    generationTimestamp: number
    basePackagingId: string
  }
}

interface TrainingDataset {
  syntheticSamples: SyntheticImage[]
  realCounterfeitSamples: ImageData[]
  syntheticRatio: number  // Should be ~0.8
  totalSamples: number
}
```

## Data Models

### Blockchain Data Structures

```typescript
// Stored on Hyperledger Fabric

interface Batch {
  batchId: string  // Primary key
  manufacturer: string  // Consortium member ID
  productionDate: number  // Unix timestamp
  expiryDate: number
  medicineName: string
  packagingSignature: number[]  // Feature vector from legitimate packaging
  currentOwner: string  // Consortium member ID
  status: 'Registered' | 'In_Transit' | 'Delivered' | 'Flagged_Counterfeit'
  registrationTxHash: string
  createdAt: number
  updatedAt: number
}

interface AuthenticityProof {
  proofId: string  // Primary key
  batchId: string  // Foreign key to Batch
  pharmacistId: string
  location: GeoLocation
  confidenceScore: number  // 0-100
  status: 'Verified_Authentic' | 'Suspicious' | 'Flagged_Counterfeit'
  scanMetadata: ScanMetadata
  voiceDescription: VoiceMetadata | null
  transactionHash: string
  linkedBatchTxHash: string  // Links to original batch registration
  timestamp: number
}

interface ScanMetadata {
  visionConfidence: number
  detectedAnomalies: Anomaly[]
  extractedBatchId: string | null
  imageCount: number
  modelVersion: string
}

interface VoiceMetadata {
  originalText: string
  originalLanguage: Language
  translatedText: string
  transcriptionConfidence: number
}

interface Transfer {
  transferId: string
  batchId: string
  fromOwner: string
  toOwner: string
  timestamp: number
  transactionHash: string
  senderSignature: string
  receiverSignature: string
}

interface CounterfeitFlag {
  flagId: string
  batchId: string
  pharmacistId: string
  location: GeoLocation
  evidence: Evidence
  timestamp: number
  transactionHash: string
  notifiedParties: string[]  // Member IDs
}

interface Evidence {
  confidenceScore: number
  anomalies: Anomaly[]
  comparisonWithLegitimate: ComparisonResult
  voiceDescription: string | null
}

interface Member {
  memberId: string  // Primary key
  role: 'Pharma_Company' | 'Rural_Pharmacist' | 'Government_Auditor'
  organizationName: string
  certificate: string  // X.509 certificate
  permissions: Permission[]
  isActive: boolean
  createdAt: number
  revokedAt: number | null
}

type Permission = 
  | 'register_batch'
  | 'transfer_batch'
  | 'verify_batch'
  | 'query_batch'
  | 'flag_counterfeit'
  | 'revoke_member'
  | 'export_reports'
  | 'view_all_transactions'
```

### Off-Chain Data Structures

```typescript
// Stored in DynamoDB or similar

interface QRCodeMapping {
  qrCodeId: string  // Primary key
  batchId: string
  transactionHash: string
  generatedAt: number
  printedOn: string  // Packaging identifier
}

interface OfflineVerificationQueue {
  queueId: string  // Primary key
  pharmacistId: string
  deviceId: string
  pendingVerifications: PreliminaryResult[]
  lastSyncAttempt: number
  syncStatus: 'pending' | 'syncing' | 'failed'
}

interface ModelCache {
  modelId: string  // Primary key
  modelType: 'vision' | 'text'
  version: string
  modelData: Buffer  // TFLite model binary
  size: number
  lastUpdated: number
}

interface BatchCache {
  deviceId: string  // Primary key
  cachedBatches: CachedBatch[]
  cacheSize: number  // Number of batches
  lastUpdated: number
  maxSize: number  // 10,000
}
```

### API Request/Response Models

```typescript
// REST API and Lambda interfaces

interface RegisterBatchRequest {
  manufacturer: string
  productionDate: string  // ISO 8601
  expiryDate: string
  medicineName: string
  packagingImages: string[]  // Base64 encoded
}

interface RegisterBatchResponse {
  batchId: string
  transactionHash: string
  qrCode: string  // Base64 encoded QR code image
  status: 'success' | 'error'
  message: string
}

interface VerifyMedicineRequest {
  images: string[]  // Base64 encoded
  voiceAudio?: string  // Base64 encoded audio
  voiceLanguage?: Language
  pharmacistId: string
  location: {
    latitude: number
    longitude: number
  }
  isOffline: boolean
}

interface VerifyMedicineResponse {
  batchId: string
  confidenceScore: number
  status: 'Verified_Authentic' | 'Suspicious' | 'Flagged_Counterfeit'
  verificationReport: {
    summary: string
    anomaliesDetected: string[]
    batchHistory: string
    recommendation: string
  }
  voiceResponse: string  // Base64 encoded audio
  transactionHash: string
}

interface QueryBatchRequest {
  batchId: string
  requesterId: string
}

interface QueryBatchResponse {
  batch: Batch
  transferHistory: Transfer[]
  verificationHistory: AuthenticityProof[]
  counterfeitFlags: CounterfeitFlag[]
}

interface DashboardQueryRequest {
  startDate: string  // ISO 8601
  endDate: string
  region?: string
  medicineType?: string
  status?: string
}

interface DashboardQueryResponse {
  statistics: CounterfeitStats
  heatMap: HeatMapData
  recentAlerts: Alert[]
}
```


## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### Batch Registration Properties

**Property 1: Batch Registration Creates Valid Blockchain Record**
*For any* valid batch registration request from an authorized Pharma_Company, the system should create a blockchain record with a unique Batch_ID, return a valid Transaction_Hash, and initialize the batch with status "Registered", owner set to the Pharma_Company, and all required metadata fields (manufacturer, production date, expiry date, medicine name, packaging signature) populated correctly.
**Validates: Requirements 1.1, 1.2, 1.3, 1.5**

**Property 2: QR Code Round-Trip**
*For any* registered batch, the generated QR code should contain the Batch_ID and Transaction_Hash, and decoding the QR code should recover these exact values.
**Validates: Requirements 1.4**

**Property 3: Batch Registration Authorization**
*For any* batch registration attempt, the system should succeed if and only if the requester is an authorized Pharma_Company member with valid credentials.
**Validates: Requirements 1.6, 12.1, 12.3**

### Multilingual Voice Processing Properties

**Property 4: Multilingual Voice Round-Trip**
*For any* voice input in a supported language (Hindi, Malayalam, Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada), the system should transcribe it to text in the source language, translate to English for processing, generate a verification result, translate the result back to the source language, and synthesize speech output in that language, preserving the semantic meaning throughout.
**Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.6, 2.7**

**Property 5: Voice Quality Error Handling**
*For any* voice input with insufficient quality for transcription (signal-to-noise ratio below threshold), the system should request the pharmacist to repeat the input rather than proceeding with unreliable transcription.
**Validates: Requirements 2.5**

**Property 6: Voice Metadata Preservation**
*For any* verification with voice input, the blockchain record should contain both the original transcribed text in the source language and the English translation, along with language metadata.
**Validates: Requirements 2.8, 7.3, 7.4, 7.5**

### Visual Analysis Properties

**Property 7: Vision Analysis Confidence Score Bounds**
*For any* packaging image analysis, the Vision_Module should generate a confidence score in the valid range [0, 100].
**Validates: Requirements 3.3**

**Property 8: Batch ID Extraction**
*For any* packaging image containing a visible Batch_ID or QR code, the Vision_Module should extract the Batch_ID correctly.
**Validates: Requirements 3.4**

**Property 9: Multi-Image Comparative Analysis**
*For any* verification with multiple images of the same package, the Vision_Module should perform comparative analysis across all images before returning results, and the confidence score should reflect the combined analysis.
**Validates: Requirements 3.10**

**Property 10: Anomaly Detection Coverage**
*For any* packaging image with known anomalies (color variations, font irregularities, hologram defects, texture mismatches), the Vision_Module should detect at least one anomaly type and reflect it in the confidence score.
**Validates: Requirements 3.1, 3.2**

### Blockchain Oracle Properties

**Property 11: Oracle Validation Against Blockchain**
*For any* completed vision analysis, the Blockchain_Oracle should query the Smart_Contract to retrieve the registered packaging signature for the detected Batch_ID before writing the authenticity proof.
**Validates: Requirements 3.5, 3.6, 9.2**

**Property 12: Authenticity Proof Completeness**
*For any* verification, the Blockchain_Oracle should write an Authenticity_Proof to the blockchain containing all required fields: Batch_ID, pharmacist identifier, location, confidence score, timestamp, scan metadata, Transaction_Hash linking to the original batch registration, and voice metadata (if voice input was provided).
**Validates: Requirements 3.7, 3.8, 6.1**

**Property 13: Confidence Score Status Mapping**
*For any* verification with confidence score >= 90, the Authenticity_Proof status should be "Verified_Authentic"; for scores < 10, status should be "Flagged_Counterfeit"; for scores in between, status should be "Suspicious".
**Validates: Requirements 3.9**

**Property 14: Oracle Orchestration Sequence**
*For any* verification request, the Blockchain_Oracle should execute components in the correct sequence: (1) Vision_Module analysis, (2) Smart_Contract query for batch history, (3) Translation_Service (if voice input), (4) generate comprehensive report combining all data, (5) write Authenticity_Proof to blockchain, (6) notify auditors if counterfeit detected.
**Validates: Requirements 9.1, 9.4, 9.5**

**Property 15: Batch Existence Validation**
*For any* verification request, the Blockchain_Oracle should validate that the scanned Batch_ID exists on the blockchain before performing detailed analysis, and should fail early if the batch does not exist.
**Validates: Requirements 9.7**

**Property 16: Historical Flag Priority Elevation**
*For any* verification of a Batch_ID that has previous counterfeit flags in its history, the Blockchain_Oracle should elevate the alert priority compared to batches with no previous flags.
**Validates: Requirements 9.3**

**Property 17: Partial Report on Component Failure**
*For any* verification where a component (Vision_Module, Translation_Service, or Smart_Contract query) fails, the Blockchain_Oracle should log the failure and generate a partial report containing all successfully retrieved data rather than failing completely.
**Validates: Requirements 9.6**

### Counterfeit Flagging Properties

**Property 18: Automatic Counterfeit Flagging**
*For any* verification with confidence score < 10, the Blockchain_Oracle should trigger a Counterfeit_Flag event in the Smart_Contract, which should update the batch status to "Flagged_Counterfeit", broadcast to all consortium members, and record the flagging pharmacist's location and identifier.
**Validates: Requirements 4.1, 4.2, 4.5**

**Property 19: Flagged Batch Transfer Prevention**
*For any* batch with status "Flagged_Counterfeit", any transfer attempt should be rejected by the Smart_Contract.
**Validates: Requirements 4.3, 8.3**

**Property 20: Counterfeit Notification Timing**
*For any* counterfeit flag event, the system should notify the Government_Auditor and the registered Pharma_Company within 60 seconds of the flag being triggered.
**Validates: Requirements 4.4**

**Property 21: Counterfeit Flag Immutability**
*For any* Counterfeit_Flag event written to the blockchain, the record should be immutable and queryable for regulatory review indefinitely.
**Validates: Requirements 4.6**

### Offline Capability Properties

**Property 22: Offline Local Inference**
*For any* verification request when internet connectivity is unavailable, the Edge_Module should perform local inference using cached models and store the result locally in a pending queue.
**Validates: Requirements 5.1, 5.2**

**Property 23: Offline Sync on Connectivity Restoration**
*For any* Edge_Module with pending offline verifications, when connectivity is restored, all pending verifications should be synchronized with the Blockchain_Oracle and written to the blockchain.
**Validates: Requirements 5.3**

**Property 24: Batch Cache Capacity**
*For any* Edge_Module, the local batch cache should maintain at least 10,000 recently registered Batch_IDs with their packaging signatures for offline lookup.
**Validates: Requirements 5.4**

**Property 25: Model and Cache Updates**
*For any* Edge_Module, when connectivity is available and new models or batch data are available on the server, the Edge_Module should update its local models and batch cache.
**Validates: Requirements 5.6**

**Property 26: Preliminary to Final Proof Transition**
*For any* offline verification, the Edge_Module should generate a preliminary Authenticity_Proof, and when connectivity is restored and sync completes, the preliminary proof should be finalized and written to the blockchain with the same verification data.
**Validates: Requirements 5.7**

### Blockchain Traceability Properties

**Property 27: Cryptographic Integrity**
*For any* record written to the blockchain, the record should have valid cryptographic signatures and endorsements according to Hyperledger Fabric's endorsement policies.
**Validates: Requirements 6.2**

**Property 28: Complete Batch History Query**
*For any* Batch_ID query, the Smart_Contract should return the complete verification history (all Authenticity_Proofs) and transfer chain (all ownership transfers) for that batch.
**Validates: Requirements 6.3, 8.4**

**Property 29: Blockchain Record Immutability**
*For any* record written to the blockchain (Batch, AuthenticityProof, Transfer, CounterfeitFlag), attempts to modify or delete the record should fail, and the original record should remain unchanged and queryable.
**Validates: Requirements 6.4**

**Property 30: Multiple Verification Preservation**
*For any* Batch_ID with multiple verifications, all verification attempts should be preserved on the blockchain with their respective timestamps and Transaction_Hash values, and querying the batch should return all verifications.
**Validates: Requirements 6.5**

**Property 31: Filtered Query Support**
*For any* query with filters (date range, location, pharmacist, verification result, consortium member), the Smart_Contract should return only records matching all specified filters.
**Validates: Requirements 6.6**

### Multilingual Communication Properties

**Property 32: English Primary Storage**
*For any* Verification_Report, the version stored on the blockchain should be in English as the primary language.
**Validates: Requirements 7.1**

**Property 33: On-Demand Report Translation**
*For any* Verification_Report and any supported language, a Government_Auditor should be able to request the report in that language and receive an accurate translation.
**Validates: Requirements 7.2**

**Property 34: Notification Translation**
*For any* notification sent by a Pharma_Company or Government_Auditor, the Translation_Service should translate the message to the recipient's preferred language before delivery.
**Validates: Requirements 7.6**

### Batch Transfer Properties

**Property 35: Transfer Ownership Verification**
*For any* batch transfer attempt, the Smart_Contract should verify the initiator has current ownership of the batch, and should reject the transfer if the initiator is not the current owner.
**Validates: Requirements 8.1**

**Property 36: Transfer State Update**
*For any* successful batch transfer, the Smart_Contract should update the batch owner to the recipient, record the transfer timestamp and Transaction_Hash, and emit a transfer event visible to all consortium members.
**Validates: Requirements 8.2, 8.5**

**Property 37: Dual Signature Authorization**
*For any* batch transfer, the system should require valid digital signatures from both the sender and receiver, and should reject transfers missing either signature.
**Validates: Requirements 8.6**

### Synthetic Data Generation Properties

**Property 38: Synthetic Anomaly Introduction**
*For any* synthetic counterfeit image generation, the system should introduce at least one realistic anomaly type (blurred text, color shift, or texture variation) into the generated image.
**Validates: Requirements 10.2**

**Property 39: Training Dataset Composition**
*For any* training dataset, the composition should maintain at least 80% synthetic data and at most 20% real counterfeit samples.
**Validates: Requirements 10.3**

**Property 40: Automatic Synthetic Generation**
*For any* new legitimate packaging design registered on the blockchain, the system should automatically generate corresponding synthetic counterfeit variations for training.
**Validates: Requirements 10.4**

**Property 41: Vision Metadata Blockchain Linking**
*For any* vision analysis result, the Rekognition metadata (confidence scores, detected features) should be linked to the Transaction_Hash of the corresponding blockchain record.
**Validates: Requirements 10.6**

### Dashboard and Analytics Properties

**Property 42: Aggregation Without Modification**
*For any* dashboard query that aggregates verification data, the individual blockchain records should remain immutable and unchanged after the aggregation.
**Validates: Requirements 11.2**

**Property 43: Suspicious Pattern Alerting**
*For any* Batch_ID with counterfeit detections across multiple geographic locations (3 or more distinct locations), the system should automatically generate an alert to Government_Auditors.
**Validates: Requirements 11.4**

**Property 44: Flagged Batch Supply Chain Display**
*For any* flagged batch, the dashboard should display the complete supply chain path showing all custody transfers from manufacturer to the point of counterfeit detection.
**Validates: Requirements 11.6**

### Authentication and Authorization Properties

**Property 45: Certificate-Based Authentication**
*For any* blockchain interaction attempt, the system should require a valid digital certificate issued by the consortium's Certificate Authority, and should reject interactions without valid certificates.
**Validates: Requirements 12.1, 12.4**

**Property 46: Member Registry Maintenance**
*For any* consortium member, the Blockchain_Network should maintain a registry entry with the member's role (Pharma_Company, Rural_Pharmacist, Government_Auditor) and associated permissions.
**Validates: Requirements 12.2**

**Property 47: Role-Based Permission Enforcement**
*For any* transaction, the Smart_Contract should verify the member's role has permission to perform the operation: Pharma_Companies can register and transfer batches, Rural_Pharmacists can verify batches, Government_Auditors have read-only access to all data.
**Validates: Requirements 12.3, 12.6**

**Property 48: Credential Revocation**
*For any* member whose credentials are revoked by a Government_Auditor, all subsequent blockchain interaction attempts by that member should be rejected.
**Validates: Requirements 12.5**


## Error Handling

### Blockchain Errors

**Transaction Failures:**
- **Endorsement Policy Violations**: If a transaction fails to meet Hyperledger Fabric's endorsement requirements, return a clear error message indicating which organizations need to endorse
- **Timeout Errors**: If blockchain write operations exceed 30 seconds, retry up to 3 times with exponential backoff before failing
- **Invalid State Errors**: If a transaction attempts to modify a batch that doesn't exist or is in an invalid state, reject with a descriptive error

**Smart Contract Errors:**
- **Authorization Failures**: Return HTTP 403 with specific permission requirements when a member attempts an unauthorized operation
- **Validation Errors**: Return HTTP 400 with field-level validation messages when batch registration or transfer data is invalid
- **Concurrency Conflicts**: Use optimistic locking with version numbers; if a concurrent modification is detected, retry the transaction

### Vision Module Errors

**Image Quality Issues:**
- **Low Resolution**: If image resolution is below 800x600, request higher quality image from pharmacist
- **Poor Lighting**: If image brightness/contrast is outside acceptable range, provide guidance for better lighting
- **Blur Detection**: If image blur exceeds threshold, request pharmacist to retake photo

**Analysis Failures:**
- **Rekognition API Errors**: If Rekognition service is unavailable, fall back to Edge_Module for offline analysis
- **Batch ID Extraction Failure**: If no Batch_ID can be extracted from images, prompt pharmacist to manually enter the Batch_ID
- **Model Inference Errors**: Log the error, notify system administrators, and provide a partial report based on available data

### Multilingual Service Errors

**Transcription Failures:**
- **Unsupported Language**: If detected language is not in the supported set, notify pharmacist and request input in a supported language
- **Low Confidence Transcription**: If transcription confidence < 70%, request pharmacist to repeat the input
- **Audio Quality Issues**: If audio has excessive noise or is too quiet, provide feedback and request clearer audio

**Translation Failures:**
- **Translation API Unavailable**: Cache the original text and retry translation when service is restored; proceed with English-only processing if urgent
- **Ambiguous Translation**: If translation confidence is low, include both original and translated text in the report with a warning flag

### Offline Mode Errors

**Cache Limitations:**
- **Batch Not in Cache**: If a scanned Batch_ID is not in the local cache during offline mode, inform pharmacist that full verification requires connectivity and provide a preliminary analysis based on visual features only
- **Cache Corruption**: If local cache integrity check fails, clear the cache and re-download when connectivity is restored
- **Storage Full**: If device storage is insufficient for pending verifications, alert pharmacist and prioritize syncing oldest pending verifications

**Sync Failures:**
- **Partial Sync**: If some verifications fail to sync, mark them for retry and continue syncing others
- **Blockchain Unavailable**: If blockchain is unreachable during sync, retry with exponential backoff (1min, 5min, 15min, 1hr)
- **Conflict Resolution**: If a batch state has changed since offline verification (e.g., batch was flagged by another pharmacist), include both verifications in the blockchain with timestamps

### Network and Infrastructure Errors

**AWS Service Outages:**
- **Bedrock Unavailable**: Fall back to rule-based orchestration logic without AI-enhanced analysis
- **Managed Blockchain Unavailable**: Queue all write operations locally and display a system-wide alert to all users
- **S3 Failures**: If image storage fails, store images locally and retry upload; proceed with verification using local images

**Certificate and Authentication Errors:**
- **Expired Certificates**: Notify member 7 days before expiration; block access after expiration until renewed
- **Revoked Certificates**: Immediately reject all operations and display revocation reason to the user
- **Invalid Signatures**: Log the attempt as a security event and notify Government_Auditors

### Data Validation Errors

**Input Validation:**
- **Invalid Batch_ID Format**: Reject with clear format requirements (e.g., "Batch ID must be alphanumeric, 12-16 characters")
- **Invalid Date Ranges**: Reject if production date > expiry date or if expiry date is in the past
- **Invalid Coordinates**: Reject if location coordinates are outside India's geographic bounds

**Business Logic Errors:**
- **Duplicate Batch Registration**: Reject if Batch_ID already exists on blockchain
- **Transfer to Self**: Reject if sender and receiver are the same member
- **Transfer of Unowned Batch**: Reject with current owner information

### Error Response Format

All API errors should follow a consistent format:

```json
{
  "error": {
    "code": "BATCH_NOT_FOUND",
    "message": "The specified Batch ID does not exist on the blockchain",
    "details": {
      "batchId": "ABC123XYZ456",
      "suggestedAction": "Verify the Batch ID on the packaging and try again"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

### Logging and Monitoring

**Error Logging:**
- All errors should be logged to CloudWatch with severity levels (ERROR, WARN, INFO)
- Include request context (user ID, batch ID, operation type) in all log entries
- Security-related errors (authentication failures, authorization violations) should trigger immediate alerts

**Monitoring Thresholds:**
- Alert if blockchain transaction failure rate exceeds 5% over 5 minutes
- Alert if Vision Module analysis time exceeds 10 seconds for 95th percentile
- Alert if offline sync queue depth exceeds 1000 pending verifications per device
- Alert if counterfeit detection rate in a region exceeds 20% over 24 hours

## Testing Strategy

### Dual Testing Approach

The system requires both unit testing and property-based testing for comprehensive coverage:

- **Unit Tests**: Verify specific examples, edge cases, error conditions, and integration points between components
- **Property Tests**: Verify universal properties across all inputs through randomized testing

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across a wide input space.

### Property-Based Testing

**Framework Selection:**
- **TypeScript/JavaScript**: Use `fast-check` library for property-based testing
- **Go (Smart Contracts)**: Use `gopter` library for property-based testing
- **Python (ML/Data Processing)**: Use `hypothesis` library for property-based testing

**Configuration:**
- Each property test MUST run a minimum of 100 iterations to ensure adequate randomization coverage
- Each property test MUST include a comment tag referencing the design document property
- Tag format: `// Feature: aushadh-verify, Property N: [property description]`

**Example Property Test:**

```typescript
// Feature: aushadh-verify, Property 1: Batch Registration Creates Valid Blockchain Record
test('batch registration creates valid blockchain record', async () => {
  await fc.assert(
    fc.asyncProperty(
      batchArbitrary(),  // Generates random valid batch data
      async (batch) => {
        const result = await registerBatch(batch);
        
        // Verify unique Batch_ID
        expect(result.batchId).toBeDefined();
        
        // Verify Transaction_Hash returned
        expect(result.transactionHash).toMatch(/^[0-9a-f]{64}$/);
        
        // Query blockchain and verify record
        const record = await queryBatch(result.batchId);
        expect(record.status).toBe('Registered');
        expect(record.currentOwner).toBe(batch.manufacturer);
        expect(record.medicineName).toBe(batch.medicineName);
        expect(record.productionDate).toBe(batch.productionDate);
        expect(record.expiryDate).toBe(batch.expiryDate);
      }
    ),
    { numRuns: 100 }
  );
});
```

**Generator Strategies:**

For effective property-based testing, implement custom generators for domain objects:

```typescript
// Batch data generator
const batchArbitrary = () => fc.record({
  manufacturer: fc.constantFrom('pharma_001', 'pharma_002', 'pharma_003'),
  productionDate: fc.date({ min: new Date('2024-01-01') }),
  expiryDate: fc.date({ min: new Date('2025-01-01'), max: new Date('2027-12-31') }),
  medicineName: fc.constantFrom('Paracetamol', 'Amoxicillin', 'Ibuprofen'),
  packagingImages: fc.array(fc.base64String(), { minLength: 1, maxLength: 5 })
});

// Voice input generator with multiple languages
const voiceInputArbitrary = () => fc.record({
  audio: fc.base64String(),
  language: fc.constantFrom('hi', 'ml', 'ta', 'te', 'bn', 'mr', 'gu', 'kn'),
  description: fc.string({ minLength: 10, maxLength: 200 })
});

// Confidence score generator
const confidenceScoreArbitrary = () => fc.integer({ min: 0, max: 100 });

// Edge cases for offline mode
const offlineVerificationArbitrary = () => fc.record({
  batchId: fc.string({ minLength: 12, maxLength: 16 }),
  images: fc.array(fc.base64String(), { minLength: 1, maxLength: 3 }),
  timestamp: fc.date(),
  isOffline: fc.constant(true)
});
```

### Unit Testing

**Component-Level Tests:**

Each component should have unit tests covering:
- Happy path scenarios
- Edge cases (empty inputs, boundary values, null/undefined)
- Error conditions (network failures, invalid data, unauthorized access)
- Integration points with other components

**Example Unit Tests:**

```typescript
describe('Vision Module', () => {
  test('should detect color variation anomaly', async () => {
    const image = loadTestImage('counterfeit_color_shift.jpg');
    const result = await visionModule.analyzePackaging([image]);
    
    expect(result.anomalies).toContainEqual(
      expect.objectContaining({ type: 'color_variation' })
    );
  });
  
  test('should handle low resolution image', async () => {
    const lowResImage = loadTestImage('low_res_400x300.jpg');
    
    await expect(
      visionModule.analyzePackaging([lowResImage])
    ).rejects.toThrow('Image resolution too low');
  });
  
  test('should extract batch ID from QR code', async () => {
    const imageWithQR = loadTestImage('packaging_with_qr.jpg');
    const batchId = await visionModule.extractBatchId(imageWithQR);
    
    expect(batchId).toBe('ABC123XYZ456');
  });
});

describe('Blockchain Oracle', () => {
  test('should trigger counterfeit flag for score < 10', async () => {
    const mockVisionResult = { confidenceScore: 5, batchId: 'TEST123' };
    const mockSmartContract = jest.fn();
    
    await blockchainOracle.verifyMedicine({
      visionResult: mockVisionResult,
      pharmacistId: 'pharm_001',
      location: { lat: 28.6139, lon: 77.2090 }
    });
    
    expect(mockSmartContract).toHaveBeenCalledWith(
      'FlagCounterfeit',
      expect.objectContaining({ batchId: 'TEST123' })
    );
  });
  
  test('should generate partial report on component failure', async () => {
    const mockVisionModule = jest.fn().mockRejectedValue(new Error('Service unavailable'));
    
    const result = await blockchainOracle.verifyMedicine({
      batchId: 'TEST123',
      images: [testImage]
    });
    
    expect(result.verificationReport.isPartial).toBe(true);
    expect(result.verificationReport.failedComponents).toContain('Vision_Module');
  });
});
```

**Smart Contract Testing:**

Use Hyperledger Fabric's testing framework for chaincode:

```go
func TestRegisterBatch(t *testing.T) {
    ctx := &MockTransactionContext{}
    contract := new(BatchContract)
    
    // Test successful registration
    txHash, err := contract.RegisterBatch(ctx, "BATCH001", "pharma_001", 
        "2024-01-01", "2026-01-01", "Paracetamol", []byte{...})
    
    assert.NoError(t, err)
    assert.NotEmpty(t, txHash)
    
    // Verify batch was stored
    batch, err := contract.QueryBatch(ctx, "BATCH001")
    assert.NoError(t, err)
    assert.Equal(t, "Registered", batch.Status)
    assert.Equal(t, "pharma_001", batch.CurrentOwner)
}

func TestTransferBatchUnauthorized(t *testing.T) {
    ctx := &MockTransactionContext{
        ClientIdentity: &MockClientIdentity{Role: "Rural_Pharmacist"},
    }
    contract := new(BatchContract)
    
    // Pharmacists should not be able to transfer batches
    _, err := contract.TransferBatch(ctx, "BATCH001", "pharma_001", "pharmacy_001", 
        []byte{...}, []byte{...})
    
    assert.Error(t, err)
    assert.Contains(t, err.Error(), "unauthorized")
}
```

### Integration Testing

**End-to-End Workflows:**

Test complete user journeys across multiple components:

```typescript
describe('Complete Verification Workflow', () => {
  test('pharmacist verifies authentic medicine with voice input', async () => {
    // 1. Register batch (as pharma company)
    const batch = await registerBatch({
      manufacturer: 'pharma_001',
      medicineName: 'Paracetamol',
      productionDate: '2024-01-01',
      expiryDate: '2026-01-01',
      packagingImages: [legitimatePackagingImage]
    });
    
    // 2. Pharmacist scans medicine
    const verificationResult = await verifyMedicine({
      images: [legitimatePackagingImage],
      voiceAudio: hindiVoiceDescription,
      voiceLanguage: 'hi',
      pharmacistId: 'pharm_001',
      location: { lat: 28.6139, lon: 77.2090 }
    });
    
    // 3. Verify results
    expect(verificationResult.status).toBe('Verified_Authentic');
    expect(verificationResult.confidenceScore).toBeGreaterThanOrEqual(90);
    expect(verificationResult.voiceResponse).toBeDefined();
    
    // 4. Verify blockchain record
    const batchHistory = await queryBatchHistory(batch.batchId);
    expect(batchHistory.verifications).toHaveLength(1);
    expect(batchHistory.verifications[0].pharmacistId).toBe('pharm_001');
  });
  
  test('offline verification syncs when connectivity restored', async () => {
    // 1. Simulate offline mode
    setNetworkConnectivity(false);
    
    // 2. Perform offline verification
    const offlineResult = await verifyMedicine({
      images: [testImage],
      pharmacistId: 'pharm_001',
      location: { lat: 28.6139, lon: 77.2090 },
      isOffline: true
    });
    
    expect(offlineResult.status).toContain('Preliminary');
    
    // 3. Restore connectivity
    setNetworkConnectivity(true);
    
    // 4. Trigger sync
    const syncResult = await edgeModule.syncWithBlockchain();
    expect(syncResult.syncedCount).toBe(1);
    
    // 5. Verify blockchain record exists
    const batchHistory = await queryBatchHistory(offlineResult.batchId);
    expect(batchHistory.verifications).toHaveLength(1);
  });
});
```

### Performance Testing

**Load Testing:**
- Simulate 1000 concurrent verification requests
- Target: 95th percentile response time < 5 seconds
- Blockchain write throughput: minimum 100 transactions per second

**Stress Testing:**
- Test offline sync with 10,000 pending verifications
- Test dashboard queries with 1 million blockchain records
- Test Vision Module with 4K resolution images

### Security Testing

**Penetration Testing:**
- Attempt unauthorized batch registration
- Attempt to modify blockchain records
- Attempt SQL injection in query parameters
- Test certificate validation and revocation

**Compliance Testing:**
- Verify GDPR compliance for personal data (pharmacist identifiers, locations)
- Verify data retention policies
- Verify audit trail completeness

### Test Coverage Goals

- **Unit Test Coverage**: Minimum 80% code coverage for all components
- **Property Test Coverage**: All 48 correctness properties must have corresponding property tests
- **Integration Test Coverage**: All critical user workflows (batch registration, verification, transfer, flagging)
- **Smart Contract Coverage**: 100% coverage of all chaincode functions

### Continuous Integration

**CI Pipeline:**
1. Run unit tests on every commit
2. Run property tests (100 iterations) on every pull request
3. Run integration tests on merge to main branch
4. Run security scans (dependency vulnerabilities, static analysis) daily
5. Deploy to staging environment for manual testing

**Test Environments:**
- **Local**: Docker Compose with local Hyperledger Fabric network
- **Staging**: AWS environment with test blockchain network
- **Production**: Full AWS Managed Blockchain consortium

