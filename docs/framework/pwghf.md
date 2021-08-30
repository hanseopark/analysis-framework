---
sort: 4
title: PWGHF
---

# Heavy-flavour (HF) analysis framework

## Contact

Coordinators: Gian Michele Innocenti, Francesco Prino, Vít Kučera

Mattermost channel: [hf-o2-analysis-challenge](https://mattermost.web.cern.ch/alice/channels/hf-o2-analysis-challenge)

## Code

* Tasks used by the heavy-flavour analysis framework are in the
[`Tasks/PWGHF`](https://github.com/AliceO2Group/O2Physics/tree/master/Tasks/PWGHF) directory.
* Tables produced by skimming and candidate creators are defined in
[`HFSecondaryVertex.h`](https://github.com/AliceO2Group/O2Physics/blob/master/DataModel/include/AnalysisDataModel/HFSecondaryVertex.h).
* Tables produced by candidate selectors are defined in
[`HFCandidateSelectionTables.h`](https://github.com/AliceO2Group/O2Physics/blob/master/DataModel/include/AnalysisDataModel/HFCandidateSelectionTables.h).
* Default parameters used in the selection of single tracks, track-index skims and candidates are defined in
[`HFSelectorCuts.h`](https://github.com/AliceO2Group/O2Physics/blob/master/Core/include/AnalysisCore/HFSelectorCuts.h).
* Secondary-vertex reconstruction algorithms are implemented in the
[`DCAFitterN`](https://github.com/AliceO2Group/AliceO2/blob/dev/Detectors/Vertexing/include/DetectorsVertexing/DCAFitterN.h) class.
* Functions for calculations of kinematic quantities and for MC matching are implemented in the
[`RecoDecay`](https://github.com/AliceO2Group/O2Physics/blob/master/Core/include/AnalysisCore/RecoDecay.h) class.
* Selection of tracks based on the particle identification (PID) detectors is performed via the
[`TrackSelectorPID`](https://github.com/AliceO2Group/O2Physics/blob/master/Core/include/AnalysisCore/TrackSelectorPID.h) class.
* Code for easy running of the HF tasks and output processing can be found in the
[Run3Analysisvalidation](https://github.com/AliceO2Group/Run3Analysisvalidation) repository.
  * Analysis code for postprocessing of the task output is collected in the
  [`FirstAnalysis`](https://github.com/AliceO2Group/Run3Analysisvalidation/tree/master/FirstAnalysis) directory.

## AliHyperloop

JIRA tickets of the HF analyses on [AliHyperloop](https://alimonitor.cern.ch/hyperloop/):
* [ALICE 2](https://alice.its.cern.ch/jira/browse/PWGHF-269)
* [ALICE 3 pp](https://alice.its.cern.ch/jira/browse/PWGHF-284)
* [ALICE 3 AA](https://alice.its.cern.ch/jira/browse/PWGHF-287)
* [ALICE 3 pp onia](https://alice.its.cern.ch/jira/browse/PWGHF-289)
* [ALICE 3 AA onia](https://alice.its.cern.ch/jira/browse/PWGHF-288)
* [DDbar correlations](https://alice.its.cern.ch/jira/browse/PWGHF-283)

## Framework structure

Simplified graph of the workflows and tasks involved in a single HF analysis is shown in the following picture.
Individual components are decribed in the next section below.

<div align="center">
<img src="../images/pwghf_graph.svg" width="100%" alt="PWGHF analysis framework">
</div>

## Framework components

### Track index skimming

Workflow | File | Type
--- | --- | ---
`o2-analysis-hf-track-index-skims-creator` | `HFTrackIndexSkimsCreator.cxx` | direct 2/3-prong and cascade decays

#### Track and event selection

Tracks and collisions are flagged with selection decisions.
Tracks selection is based on reconstruction quality and kinematic criteria.
Collisions are selected based on the required triggers.

#### Track combination and protocandidate preselection

Double and triple loops over the selected tracks in each selected event are performed to combine tracks into *protocandidates*.
Very loose selection criteria are applied (invariant mass and *p*<sub>T</sub> of the protocandidate) to reject fake candidates before secondary-vertex finding.
Secondary vertex is reconstructed for the selected track pairs and triplets and the tracks are propagated to the decay vertex.
A first selection at the candidate level is applied (candidate *p*<sub>T</sub>, cosine of pointing angle, product of prong impact parameters).

Indices of the daughter tracks of selected candidates are stored as columns of a new derived table (*track index skim table*),
together with a flag indicating for which decay channel(s) the candidate was selected.

### Candidate creation and MC matching

Workflow | File | Type
--- | --- | ---
`o2-analysis-hf-candidate-creator-2prong` | `HFCandidateCreator2Prong.cxx` | direct 2-prong decays
`o2-analysis-hf-candidate-creator-3prong` | `HFCandidateCreator3Prong.cxx` | direct 3-prong decays
`o2-analysis-hf-candidate-creator-cascade` | `HFCandidateCreatorCascade.cxx` | cascade decays with V<sup>0</sup> daughters (Λ<sub>c</sub><sup>±</sup>)
`o2-analysis-hf-candidate-creator-xicc` | `HFCandidateCreatorXicc.cxx` | Ξ<sub>cc</sub><sup>±±</sup> → Ξ<sub>c</sub><sup>±</sup> π<sup>±</sup>
`o2-analysis-hf-candidate-creator-x` | `HFCandidateCreatorX.cxx` | X(3872) → J/ψ π<sup>±</sup> π<sup>∓</sup>

#### Candidate creation

Track indices in the track index skim table are used to build full decay candidates.
The secondary-vertex reconstruction is repeated and additional candidate properties,
that are needed for the signal selection and cannot be calculated dynamically on-the-fly,
are computed (e.g. uncertainties of quantities computed in the vertex reconstruction procedure).

The complete list of quantities needed for the final candidate selection and analysis are stored
in a derived table of reconstructed candidates (*candidate table*).

#### MC matching

For simulated data, reconstructed decay candidates are matched with their generated counterparts
by checking the correspondence between the candidate prongs and the expected decay tree.
The MC matching procedure is performed also for generated MC particles by checking their identity and their decay tree.
Particle origin is determined by inspecting the decay tree to identify non-prompt particles, produced from b quarks.

Derived tables with MC flags used for the estimation of the signal efficiencies and the optimisation of the signal and background selections are produced.

### Candidate selection

Workflow | File | Type
--- | --- | ---
`o2-analysis-hf-d0-candidate-selector` | `HFD0CandidateSelector.cxx` | D<sup>0</sup>(bar) → π<sup>±</sup> K<sup>∓</sup>
`o2-analysis-hf-jpsi-candidate-selector` | `HFJpsiCandidateSelector.cxx` | J/ψ → e<sup>+</sup> e<sup>−</sup>/μ<sup>+</sup> μ<sup>−</sup>
`o2-analysis-hf-x-tojpsipipi-candidate-selector` | `HFXToJpsiPiPiCandidateSelector.cxx` | X(3872) → J/ψ π<sup>±</sup> π<sup>∓</sup>
`o2-analysis-hf-dplus-topikpi-candidate-selector` | `HFDplusToPiKPiCandidateSelector.cxx` | D<sup>±</sup> → π<sup>±</sup> K<sup>∓</sup> π<sup>±</sup>
`o2-analysis-hf-lc-candidate-selector` | `HFLcCandidateSelector.cxx` | Λ<sub>c</sub><sup>±</sup> → p(bar) K<sup>∓</sup> π<sup>±</sup>
`o2-analysis-hf-lc-tok0sp-candidate-selector` | `HFLcK0sPCandidateSelector.cxx` | Λ<sub>c</sub><sup>±</sup> → p(bar) K<sub>S</sub><sup>0</sup>
`o2-analysis-hf-xic-topkpi-candidate-selector` | `HFXicToPKPiCandidateSelector.cxx` | Ξ<sub>c</sub><sup>±</sup> → p(bar) K<sup>∓</sup> π<sup>±</sup>
`o2-analysis-hf-xicc-topkpipi-candidate-selector` | `HFXiccToPKPiPiCandidateSelector.cxx` | Ξ<sub>cc</sub><sup>±±</sup> → Ξ<sub>c</sub><sup>±</sup> π<sup>±</sup>

In a dedicated selector task, tailored for each decay channel, accurate analysis level selection criteria
based on decay topology and PID are applied to the reconstructed candidates.

The selection results are stored in a column of a new dedicated table that is later joined with the candidate table to filter them.

### Analysis task

Workflow | File | Type
--- | --- | ---
`o2-analysis-hf-task-d0` | `taskD0.cxx` | D<sup>0</sup>(bar) → π<sup>±</sup> K<sup>∓</sup>
`o2-analysis-hf-task-jpsi` | `taskJpsi.cxx` | J/ψ → e<sup>+</sup> e<sup>−</sup>/μ<sup>+</sup> μ<sup>−</sup>
`o2-analysis-hf-task-dplus` | `taskDPlus.cxx` | D<sup>±</sup> → π<sup>±</sup> K<sup>∓</sup> π<sup>±</sup>
`o2-analysis-hf-task-lc` | `taskLc.cxx` | Λ<sub>c</sub><sup>±</sup> → p(bar) K<sup>∓</sup> π<sup>±</sup>
`o2-analysis-hf-task-lc-tok0sp` | `taskLcK0sP.cxx` | Λ<sub>c</sub><sup>±</sup> → p(bar) K<sub>S</sub><sup>0</sup>
`o2-analysis-hf-task-xic` | `taskXic.cxx` | Ξ<sub>c</sub><sup>±</sup> → p(bar) K<sup>∓</sup> π<sup>±</sup>
`o2-analysis-hf-task-xicc` | `taskXicc.cxx` | Ξ<sub>cc</sub><sup>±±</sup> → Ξ<sub>c</sub><sup>±</sup> π<sup>±</sup>
`o2-analysis-hf-task-bplus` | `taskBPlus.cxx` | B<sup>±</sup> → D<sup>0</sup>(bar) π<sup>±</sup>
`o2-analysis-hf-task-x` | `taskX.cxx` | X(3872) → J/ψ π<sup>±</sup> π<sup>∓</sup>
`o2-analysis-hf-hf-correlator-d0d0bar` | `HFCorrelatorD0D0bar.cxx` | D<sup>0</sup>–D<sup>0</sup>bar correlations
`o2-analysis-hf-hf-correlator-dplusdminus` | `HFCorrelatorDplusDminus.cxx` | D<sup>+</sup>–D<sup>−</sup> correlations
`o2-analysis-hf-task-correlation-ddbar` | `taskCorrelationDDbar.cxx` | D<sup>0</sup>–D<sup>0</sup>bar, D<sup>+</sup>–D<sup>−</sup> correlations

#### Real-data analysis

In the user analysis task, histograms needed for the analysis are filled with properties of selected candidates.

#### MC-data analysis

For MC events, histograms with quantities of generated MC particles and MC-matched candidates are produced.

### Tree creation

Candidate tables and other related derived tables are exported to disk as ROOT trees for
post-processing with external tools, e.g. for optimisation with Machine Learning techniques.

Workflow | File | Type
--- | --- | ---
`o2-analysis-hf-tree-creator-d0-tokpi` | `HFTreeCreatorD0ToKPi.cxx` | D<sup>0</sup>(bar) → π<sup>±</sup> K<sup>∓</sup>
`o2-analysis-hf-tree-creator-lc-topkpi` | `HFTreeCreatorLcToPKPi.cxx` | Λ<sub>c</sub><sup>±</sup> → p(bar) K<sup>∓</sup> π<sup>±</sup>

### QA

Distributions of various quantities are saved in histograms by dedicated tasks to validate code and data.

Workflow | File | Type
--- | --- | ---
`o2-analysis-hf-mc-validation` | `HFMCValidation.cxx` | validation of HF MC distributions
`o2-analysis-qa-rejection` | `qaPidRejection.cxx` | PID selection performance