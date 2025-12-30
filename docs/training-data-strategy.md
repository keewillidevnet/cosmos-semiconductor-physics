# Training Data Strategy

## Cosmos for Semiconductor Physics (CSP)

---

## 1. The Data Challenge

Training a physics-native foundation model for semiconductor fabrication requires overcoming significant data barriers:

| Challenge | Description | Mitigation |
|-----------|-------------|------------|
| **Proprietary data** | Fabs guard process recipes as core IP | Multi-source strategy, synthetic data |
| **Fragmentation** | Data scattered across systems | Unified data model |
| **Inconsistency** | Different fabs use different conventions | Normalization pipelines |
| **Tribal knowledge** | Much expertise is undocumented | Expert interviews, literature mining |

---

## 2. Multi-Source Data Strategy

CSP leverages four complementary data sources:

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA SOURCE HIERARCHY                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  TIER 1: FIRST-PRINCIPLES SIMULATIONS (Synthetic)           │   │
│  │  • Ground truth physics                                      │   │
│  │  • No IP concerns                                            │   │
│  │  • Systematic coverage                                       │   │
│  │  Volume: 10⁷ - 10⁸ simulations                              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  TIER 2: RESEARCH FAB DATA (Semi-Public)                    │   │
│  │  • Real process data                                         │   │
│  │  • Accessible via partnerships                               │   │
│  │  • Diverse process conditions                                │   │
│  │  Volume: 10⁵ - 10⁶ wafer lots                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  TIER 3: PRODUCTION FAB DATA (TSMC Partnership)             │   │
│  │  • Leading-edge process data                                 │   │
│  │  • High-volume manufacturing insights                        │   │
│  │  • Requires trust relationship                               │   │
│  │  Volume: Selective, high-value                              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  TIER 4: PUBLISHED LITERATURE (Public)                      │   │
│  │  • Process recipes (often incomplete)                        │   │
│  │  • Device characteristics                                    │   │
│  │  • Failure analysis                                          │   │
│  │  Volume: 100,000+ papers                                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Tier 1: First-Principles Simulations

### 3.1 Simulation Types

| Simulation | Physics Covered | Tools | Data Volume |
|------------|-----------------|-------|-------------|
| **TCAD** | Device physics, process simulation | Sentaurus, Silvaco, Lumerical | 10⁶-10⁸ |
| **DFT** | Electronic structure, material properties | VASP, Quantum ESPRESSO | 10⁵-10⁷ |
| **Molecular Dynamics** | Atomic-scale processes, interfaces | LAMMPS, GROMACS | 10⁴-10⁶ |
| **Monte Carlo** | Ion transport, stochastic lithography | Custom, SRIM | 10⁶-10⁸ |
| **FDTD/EM** | Lithography optics, interconnect | Lumerical, COMSOL | 10⁵-10⁷ |
| **FEA** | Stress, thermal, reliability | ANSYS, COMSOL | 10⁵-10⁶ |

### 3.2 TCAD Data Generation Pipeline

```python
class TCADDataPipeline:
    """Systematic TCAD simulation for training data generation"""
    
    def __init__(self, simulator='sentaurus'):
        self.simulator = TCADSimulator(simulator)
        self.param_sampler = LatinHypercubeSampler()
        
    def generate_device_dataset(self, device_type, n_samples):
        """Generate device simulation dataset"""
        
        # Define parameter space
        param_space = self.get_param_space(device_type)
        
        # Sample parameters using Latin Hypercube
        params = self.param_sampler.sample(param_space, n_samples)
        
        results = []
        for p in params:
            # Run simulation
            sim_result = self.simulator.run_device(
                device_type=device_type,
                geometry=p['geometry'],
                doping=p['doping'],
                materials=p['materials']
            )
            
            results.append({
                'params': p,
                'iv_curves': sim_result.iv,
                'cv_curves': sim_result.cv,
                'band_structure': sim_result.bands,
                'carrier_distribution': sim_result.carriers
            })
            
        return results
    
    def generate_process_dataset(self, process_type, n_samples):
        """Generate process simulation dataset"""
        
        param_space = self.get_process_param_space(process_type)
        params = self.param_sampler.sample(param_space, n_samples)
        
        results = []
        for p in params:
            sim_result = self.simulator.run_process(
                process_type=process_type,
                recipe=p['recipe'],
                incoming_structure=p['structure']
            )
            
            results.append({
                'params': p,
                'output_structure': sim_result.structure,
                'profile': sim_result.profile,
                'uniformity': sim_result.uniformity
            })
            
        return results
```

### 3.3 Parameter Space Coverage

Systematic exploration ensures training data covers:

- **Normal operating conditions**: Production-relevant parameters
- **Edge cases**: Near equipment limits
- **Failure modes**: Out-of-spec conditions
- **Novel regions**: Unexplored parameter combinations

```python
class ParameterSpaceExplorer:
    """Ensures comprehensive parameter space coverage"""
    
    def __init__(self):
        self.coverage_tracker = CoverageTracker()
        
    def adaptive_sampling(self, param_space, current_data, n_new):
        """Sample to fill coverage gaps"""
        
        # Identify under-sampled regions
        gaps = self.coverage_tracker.find_gaps(
            param_space, 
            current_data
        )
        
        # Prioritize novel regions
        priority_regions = self.prioritize_for_discovery(gaps)
        
        # Sample from priority regions
        new_samples = self.sample_regions(
            priority_regions, 
            n_new
        )
        
        return new_samples
```

---

## 4. Tier 2: Research Fab Partnerships

### 4.1 Target Institutions

| Institution | Location | Capabilities | Access Path |
|-------------|----------|--------------|-------------|
| **IMEC** | Belgium | Advanced nodes, all foundries collaborate | Membership program |
| **Albany NanoTech** | USA | 300mm research fab, EUV | NY CREATES partnership |
| **Leti** | France | Advanced packaging, photonics | Collaborative R&D |
| **AIST** | Japan | Wide bandgap, MEMS | Academic collaboration |
| **Fraunhofer** | Germany | Heterointegration, power devices | Joint projects |

### 4.2 Data Types Available

Research fabs can provide:

- Process recipes (with publication restrictions)
- Metrology data (CD-SEM, OCD, XRD)
- Electrical test data
- Defect inspection results
- Process-structure correlations

### 4.3 Partnership Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    RESEARCH FAB PARTNERSHIP                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  NVIDIA Provides:              Research Fab Provides:           │
│  ┌─────────────────────┐      ┌─────────────────────┐          │
│  │ • GPU compute       │      │ • Process data      │          │
│  │ • Model development │◀────▶│ • Fab access        │          │
│  │ • ML expertise      │      │ • Domain expertise  │          │
│  │ • Publication rights│      │ • Validation runs   │          │
│  └─────────────────────┘      └─────────────────────┘          │
│                                                                 │
│  Joint Outcomes:                                                │
│  • Co-authored publications                                     │
│  • Shared model improvements                                    │
│  • Process innovations (IP shared or split)                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Tier 3: TSMC Partnership

### 5.1 Existing Infrastructure

The cuLitho collaboration has established:

- Data transfer protocols
- Security/confidentiality frameworks
- Joint engineering teams
- Validation workflows

### 5.2 Data Access Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    TSMC DATA ACCESS MODEL                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Phase 1: Aggregated/Anonymized                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Statistical distributions (not individual recipes)    │   │
│  │ • Process windows and margins                           │   │
│  │ • Yield correlations (aggregated)                       │   │
│  │ • Defect statistics                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  Phase 2: Federated Learning                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Model trained on TSMC premises                        │   │
│  │ • Only model weights transferred (not data)             │   │
│  │ • TSMC maintains data sovereignty                       │   │
│  │ • Joint validation on held-out data                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  Phase 3: Secure Enclave (Future)                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Confidential computing environment                    │   │
│  │ • Data never leaves encrypted enclave                   │   │
│  │ • Cryptographic verification of computations            │   │
│  │ • Full data access with privacy guarantees              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 Data Categories

| Category | Sensitivity | Access Phase |
|----------|-------------|--------------|
| Metrology statistics | Medium | Phase 1 |
| Process windows | Medium | Phase 1 |
| Yield correlations | High | Phase 2 |
| Recipe parameters | Critical | Phase 2-3 |
| Device data | Critical | Phase 3 |

---

## 6. Tier 4: Literature Mining

### 6.1 Sources

- **IEEE Xplore**: IEDM, VLSI Symposium, TED papers
- **arXiv**: Preprints on semiconductor physics/ML
- **Patent databases**: USPTO, EPO for process innovations
- **Conference proceedings**: ECS, MRS, SPIE

### 6.2 Extraction Pipeline

```python
class LiteratureDataPipeline:
    """Extract structured data from semiconductor publications"""
    
    def __init__(self):
        self.pdf_parser = PDFParser()
        self.ner_model = SemiconductorNER()  # Named entity recognition
        self.relation_extractor = RelationExtractor()
        
    def process_paper(self, pdf_path):
        """Extract structured information from paper"""
        
        # Parse PDF
        text, figures, tables = self.pdf_parser.parse(pdf_path)
        
        # Extract entities
        entities = self.ner_model.extract(text)
        # Entities: materials, processes, parameters, metrics
        
        # Extract relations
        relations = self.relation_extractor.extract(text, entities)
        # Relations: process→outcome, parameter→effect
        
        # Extract from tables
        table_data = self.extract_tables(tables)
        
        # Extract from figures (if possible)
        figure_data = self.extract_figures(figures)
        
        return {
            'entities': entities,
            'relations': relations,
            'tables': table_data,
            'figures': figure_data
        }
```

### 6.3 Data Quality

Literature data requires careful handling:

- **Incomplete recipes**: Papers often omit details
- **Inconsistent units**: Standardization needed
- **Conflicting results**: Different labs, conditions
- **Publication bias**: Failures rarely reported

---

## 7. Data Representation

### 7.1 Unified Data Model

```python
@dataclass
class ProcessStep:
    """Unified representation of a process step"""
    step_type: str  # 'deposition', 'etch', 'litho', etc.
    parameters: Dict[str, float]
    materials: List[Material]
    equipment: str
    incoming_structure: Structure3D
    outgoing_structure: Structure3D
    metrology: Dict[str, Measurement]
    
@dataclass
class ProcessFlow:
    """Complete process flow"""
    steps: List[ProcessStep]
    target_device: DeviceSpec
    yield_data: Optional[YieldData]
    source: str  # 'simulation', 'research_fab', 'production', 'literature'
    confidence: float
```

### 7.2 Data Normalization

All data sources normalized to common format:

```python
class DataNormalizer:
    """Normalize data from different sources"""
    
    def normalize_units(self, data):
        """Convert all units to SI base"""
        pass
        
    def normalize_coordinates(self, data):
        """Standardize coordinate systems"""
        pass
        
    def normalize_materials(self, data):
        """Map material names to standard identifiers"""
        pass
        
    def estimate_confidence(self, data, source):
        """Assign confidence based on source and completeness"""
        pass
```

---

## 8. Data Volume Estimates

| Data Type | Tier 1 (Sim) | Tier 2 (Research) | Tier 3 (TSMC) | Tier 4 (Lit) |
|-----------|--------------|-------------------|---------------|--------------|
| Process steps | 10⁸ | 10⁶ | 10⁷ | 10⁵ |
| Device simulations | 10⁷ | 10⁵ | 10⁶ | 10⁴ |
| Full flows | 10⁵ | 10⁴ | 10⁵ | 10³ |
| Total tokens | ~10¹² | ~10¹⁰ | ~10¹¹ | ~10⁹ |

---

## 9. Data Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DATA PIPELINE ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │   TCAD   │  │ Research │  │   TSMC   │  │Literature│           │
│  │Simulations│  │   Fabs   │  │(Federated)│  │  Mining  │           │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘           │
│       │             │             │             │                   │
│       └─────────────┴──────┬──────┴─────────────┘                   │
│                            │                                        │
│                     ┌──────▼──────┐                                 │
│                     │    Data     │                                 │
│                     │   Ingestion │                                 │
│                     └──────┬──────┘                                 │
│                            │                                        │
│                     ┌──────▼──────┐                                 │
│                     │   Quality   │                                 │
│                     │   Control   │                                 │
│                     └──────┬──────┘                                 │
│                            │                                        │
│                     ┌──────▼──────┐                                 │
│                     │Normalization│                                 │
│                     │  & Fusion   │                                 │
│                     └──────┬──────┘                                 │
│                            │                                        │
│                     ┌──────▼──────┐                                 │
│                     │  Tokenizer  │                                 │
│                     │             │                                 │
│                     └──────┬──────┘                                 │
│                            │                                        │
│                     ┌──────▼──────┐                                 │
│                     │  Training   │                                 │
│                     │   Dataset   │                                 │
│                     └─────────────┘                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

*For technical details on how this data is used in training, see [technical-architecture.md](technical-architecture.md)*
