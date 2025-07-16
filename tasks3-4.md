# Notes for LLM Conversations

### What a Task Should Entail:

- **Clear Problem Statement**: Define the specific issue or goal that needs to be addressed.
- **Context and Background**: Provide relevant information about why this task is needed and any recent changes that led to it.
- **Best Solution Recommendation**: Suggest the most effective approach to solve the problem, with rationale.
- **Sub-Tasks**: Break down the task into manageable, sequential steps that build upon each other.

### What a Sub-Task Should Entail:

- **Specific Action Items**: Clear, actionable steps that can be completed in a single LLM conversation.
- **Verification Requirements**: Each sub-task must include testing/verification steps to ensure the work is complete and functional.
- **Dependencies**: Sub-tasks should be designed so that each one produces tools/functions that are ready to be used by subsequent sub-tasks.
- **Success Criteria**: Clear definition of what "done" means for each sub-task.

### Verification Requirements:

- **Functionality Testing**: Verify that any new functions or tools work as expected.
- **Integration Testing**: Ensure that refactored code integrates properly with existing systems.
- **Edge Case Handling**: Test unusual scenarios (e.g., missing files, different working directories).
- **Documentation Validation**: Verify that any documentation or examples actually work.

_Each sub-task is actionable and can be completed in a single LLM prompt, but the overall task is best done in sequence for maintainability and reliability. Each sub-task must be fully verified before moving to the next one._

### IMPORTANT Notes:

- When updating any documentation (including sub-tasks) about the information about a completed sub-task, ensure to include all errors that occured, what their problem was, and how they were fixed.

---

# Task 3: Interactive Dashboard for Sliding Window Event Detection Tool

## Issue: Manual and Time-Consuming Event Detection Workflow

The current sliding window event detection process is manual, requiring users to run command-line scripts for each step (ABF conversion, preprocessing, event detection) and manually review results. This workflow lacks interactivity, makes it difficult to iterate on parameters, and provides no way to flag and organize detected events. Users need to manually zoom in/out on traces to verify events, which is time-consuming and error-prone. There's no systematic way to manage multiple analyses, compare results, or export findings in a user-friendly format.

## Best Solution: Interactive Streamlit Dashboard with Complete Pipeline Integration

The most effective approach is to create a comprehensive Streamlit dashboard that integrates the entire event detection pipeline into a single, interactive web application. The dashboard should:

- **File Upload & Processing**: Handle ABF file uploads and process them through the complete pipeline
- **Interactive Configuration**: Allow users to configure preprocessing and detection parameters with real-time validation
- **Model Selection**: Integrate with the model registry to select appropriate models with compatibility warnings
- **Real-time Processing**: Provide progress tracking and status updates during processing
- **Interactive Event Review**: Enable zoom, pan, and flagging of detected events with notes
- **History Management**: Track analysis sessions and allow re-analysis of previous files
- **Export Capabilities**: Export flagged events and analysis results in multiple formats

This approach will transform the manual command-line workflow into an intuitive, interactive experience that accelerates event detection and improves accuracy through better visualization and organization.

## Context: Current Tools and Recent Work

### Task 1 & 2 Completion Context:

**Tasks 1 and 2 were successfully completed with the following key accomplishments:**

1. **Project Root Utility** (`src/utils/paths.py`): Robust path resolution system that works from any location
2. **Model Registry** (`scripts/analysis/core/model_registry.py`): Comprehensive model discovery and metadata management
3. **Test Configuration Management** (`scripts/analysis/core/test_configs.py`): Preset configurations for different evaluation scenarios
4. **Enhanced Batch Testing** (`scripts/analysis/core/batch_test_models.py`): Advanced batch testing with parallel execution
5. **Model Comparison Tools** (`scripts/analysis/core/model_comparison.py`): Side-by-side model analysis and visualization
6. **Analysis Dashboard** (`scripts/analysis/core/dashboard.py`): Interactive dashboard for model evaluation results

**Key Infrastructure Available:**

- **Robust Path Management**: All scripts use project root-based paths for reliability
- **Model Registry Integration**: Automatic model discovery and metadata extraction
- **Configuration Management**: Preset configurations with validation and customization
- **Progress Tracking**: Real-time progress monitoring and status updates
- **Data Export**: Multiple format support (CSV, JSON, Excel)

### Current Sliding Window Tool:

**`scripts/model_event_extraction/model_event_extraction.py`:**

- Takes preprocessed .npy files as input
- Applies sliding window detection with configurable parameters
- Outputs event metadata and extraction summaries
- Supports `--no_save_segments` flag for dashboard integration

### Preprocessing Pipeline:

**ABF to NPY Conversion** (`scripts/preprocessing/abf_to_npy_data_conversion.py`):

- Converts ABF files to NumPy format with [time(s), current(pA)] structure
- Supports parallel processing for multiple files

**Main Preprocessing** (`scripts/preprocessing/main_preprocessing.py`):

- Baseline correction, thresholding, low-pass filtering
- Savitzky-Golay smoothing, downsampling, normalization
- Configurable parameters with validation

### Current Limitations:

- **Manual Workflow**: Each step requires separate command-line execution
- **No Interactive Review**: Events must be verified by manually examining plots
- **Limited Parameter Iteration**: Changing parameters requires re-running entire pipeline
- **No Event Organization**: No way to flag, categorize, or annotate detected events
- **No History Management**: Previous analyses are not systematically tracked
- **Poor User Experience**: Command-line interface is not user-friendly for non-technical users

### Critical Design Specifications

**📋 Detailed Design Information:**
For comprehensive UI/UX design, navigation flow, database schema, technical specifications, and implementation details, refer to:
**`draft/notes/task3_progress.md`**

This document contains:

- **Complete UI/UX Design**: Detailed page layouts, navigation flow, and user experience specifications
- **Database Schema**: Full CSV storage structure for sessions, events, and processing logs
- **Technical Architecture**: Data flow diagrams, integration points, and performance requirements
- **Implementation Guidelines**: Detailed specifications for each dashboard component
- **Visual Mockups**: ASCII art layouts and mermaid diagrams for all dashboard pages

---

## Sub-Tasks for Task 3

### 3A. Create Core Streamlit Application Structure

- Create the main Streamlit application file (`scripts/dashboard/event_detection_dashboard.py`) with basic navigation structure
- Implement session state management for tracking user progress and data persistence
- Set up the project root integration and import patterns following Task 1's established patterns
- Create basic page layout with navigation sidebar and main content area
- **VERIFICATION**: Test the basic Streamlit app runs without errors and displays the navigation structure
- **VERIFICATION**: Verify session state management works correctly for storing and retrieving user data
- **VERIFICATION**: Test the app works from different directories using project root integration

### 3B. Implement File Upload and Validation System

- Create file upload interface for ABF files with size and format validation
- Implement file information extraction (duration, sampling rate, file size) using PyABF
- Add file preview functionality showing basic trace information
- Create temporary file management system for uploaded files
- **VERIFICATION**: Test file upload with various ABF file sizes and formats
- **VERIFICATION**: Verify file information extraction works correctly for different ABF files
- **VERIFICATION**: Test error handling for invalid files, corrupted data, and unsupported formats
- **VERIFICATION**: Confirm temporary file cleanup works properly after session ends

### 3C. Create Preprocessing Configuration Interface

- Build interactive configuration interface for preprocessing parameters with real-time validation
- Implement parameter presets (default, high-quality, fast-processing) with custom override options
- Add parameter validation to ensure values are within acceptable ranges
- Create configuration persistence and loading functionality
- **VERIFICATION**: Test all preprocessing parameters can be configured and validated correctly
- **VERIFICATION**: Verify preset configurations load and apply correctly
- **VERIFICATION**: Test parameter validation catches invalid values and provides helpful error messages
- **VERIFICATION**: Confirm configuration persistence works across browser sessions

### 3D. Integrate Model Registry and Selection Interface

- Create model selection interface that queries the model registry for available models
- Implement model filtering by type, performance metrics, and training date
- Add model compatibility checking that compares preprocessing parameters with model training configuration
- Display model metadata and performance metrics in an organized format
- **VERIFICATION**: Test model registry integration and verify all available models are displayed
- **VERIFICATION**: Verify model filtering works correctly for all filter criteria
- **VERIFICATION**: Test compatibility checking identifies mismatches between preprocessing and model training
- **VERIFICATION**: Confirm model metadata display shows all relevant information clearly

### 3E. Implement Processing Pipeline Integration

- Integrate ABF to NPY conversion with progress tracking and error handling
- Integrate preprocessing pipeline with real-time progress updates and status display
- Integrate sliding window detection with progress monitoring and intermediate results
- Implement comprehensive error handling and recovery mechanisms
- **VERIFICATION**: Test complete pipeline integration with various ABF files
- **VERIFICATION**: Verify progress tracking provides accurate updates for each processing step
- **VERIFICATION**: Test error handling gracefully manages processing failures and provides recovery options
- **VERIFICATION**: Confirm intermediate results are preserved and can be resumed after interruptions

### 3F. Create Interactive Event Visualization System

- Implement Plotly-based interactive event visualization with zoom, pan, and hover capabilities
- Create event navigation system (previous/next, jump to specific events)
- Add event filtering and sorting capabilities (by confidence, time, duration)
- Implement event highlighting and selection functionality
- **VERIFICATION**: Test interactive visualization with various event datasets
- **VERIFICATION**: Verify event navigation works smoothly for large numbers of events
- **VERIFICATION**: Test filtering and sorting functionality with different criteria
- **VERIFICATION**: Confirm event highlighting and selection provides clear visual feedback

### 3G. Implement Event Flagging and Annotation System

- Create event flagging interface (confirm event, mark as non-event, skip)
- Implement notes system for adding user annotations to events
- Add batch flagging capabilities for processing multiple events at once
- Create flagging statistics and progress tracking
- **VERIFICATION**: Test individual event flagging with various event types
- **VERIFICATION**: Verify notes system allows adding and editing annotations
- **VERIFICATION**: Test batch flagging functionality with different selection methods
- **VERIFICATION**: Confirm flagging statistics update correctly and provide useful feedback

### 3H. Build Session History and Management System

- Create session persistence system that saves analysis sessions to CSV files
- Implement history interface for viewing and managing previous analyses
- Add session filtering and search capabilities
- Create session re-analysis functionality
- **VERIFICATION**: Test session persistence saves all relevant data correctly
- **VERIFICATION**: Verify history interface displays sessions in an organized manner
- **VERIFICATION**: Test filtering and search functionality with various criteria
- **VERIFICATION**: Confirm re-analysis functionality works correctly for previous sessions

### 3I. Implement Export and Data Management System

- Create export functionality for flagged events in multiple formats (CSV, JSON, Excel)
- Implement analysis summary export with processing parameters and statistics
- Add data cleanup utilities for managing temporary files and old sessions
- Create export customization options (include metadata, processing parameters, user notes)
- **VERIFICATION**: Test export functionality with various data formats and configurations
- **VERIFICATION**: Verify analysis summaries contain all relevant information
- **VERIFICATION**: Test data cleanup removes temporary files without affecting important data
- **VERIFICATION**: Confirm export customization options work correctly for all supported formats

### 3J. Add Advanced Features and User Experience Enhancements

- Implement real-time processing status updates with estimated completion times
- Add comprehensive error handling with user-friendly error messages and recovery suggestions
- Create help system with tooltips, documentation, and best practices
- Implement performance optimization for large files and datasets
- **VERIFICATION**: Test real-time status updates provide accurate progress information
- **VERIFICATION**: Verify error handling provides clear, actionable error messages
- **VERIFICATION**: Test help system provides useful information for all dashboard features
- **VERIFICATION**: Confirm performance optimization handles large files efficiently

### 3K. Create Comprehensive Testing and Documentation

- Develop comprehensive test suite covering all dashboard functionality
- Create user documentation with usage examples and troubleshooting guide
- Implement automated testing for critical user workflows
- Add performance benchmarking and optimization validation
- **VERIFICATION**: Test all dashboard features work correctly across different scenarios
- **VERIFICATION**: Verify documentation is clear, complete, and includes working examples
- **VERIFICATION**: Test automated workflows handle edge cases and error conditions
- **VERIFICATION**: Confirm performance meets requirements for large datasets and multiple users

### 3L. Polish and Final Integration

- Implement final UI/UX polish with consistent styling and branding
- Add final error handling and edge case management
- Create deployment configuration and requirements management
- Implement final testing and validation of complete system
- **VERIFICATION**: Test complete dashboard workflow from file upload to export
- **VERIFICATION**: Verify all features work together seamlessly
- **VERIFICATION**: Test dashboard handles all edge cases and error conditions gracefully
- **VERIFICATION**: Confirm deployment configuration works correctly in target environment

---

# Task 4: [Future Task - To Be Defined]

## Issue: [To be defined based on project needs]

## Best Solution: [To be defined based on analysis]

## Context: [To be defined based on current state]

---

## Sub-Tasks for Task 4

### 4A. [To be defined]

### 4B. [To be defined]

### 4C. [To be defined]

### 4D. [To be defined]

### 4E. [To be defined]

### 4F. [To be defined]

### 4G. [To be defined]

### 4H. [To be defined]

### 4I. [To be defined]

### 4J. [To be defined]

### 4K. [To be defined]

### 4L. [To be defined]
