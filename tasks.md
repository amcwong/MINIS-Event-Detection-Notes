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

# Task 1: Make File Access Robust to Project Structure Changes

## Issue: Fragile Relative Paths in Scripts

Many scripts in this project access data, models, or other scripts using relative paths (e.g., `../`, `../../`). This approach is fragile: when scripts are moved, or the project structure changes, these paths break, causing import errors, file-not-found errors, and general maintenance headaches. This has been a recurring source of bugs, especially after the recent reorganization of analysis, verification, and visualization scripts into new directories.

## Best Solution: Use a Project Root Anchor with Configurable Paths

The most robust solution is to anchor all file and directory access to a single, well-defined project root. Scripts should:

- Dynamically determine the project root (e.g., by searching for a marker file like `README.md`, `.git`, or a config file), or
- Use an environment variable (e.g., `PROJECT_ROOT`) set at runtime, or
- Use a central configuration file (e.g., `config.py`, `config.yaml`) that defines all key paths relative to the project root.

All file accesses should then be constructed relative to this root, not the script's own location. This makes the codebase robust to file moves and directory reorganizations.

## Context: Recent Moving Changes and Path Issues

- Scripts were recently moved from `scripts/event_verification/` and `scripts/event_verification/visualization/` to `scripts/verification/` and `scripts/visualization/`.
- Many scripts still use paths like `../../../data/labels/c57-labelled-data`, which break when the script is moved.
- Some scripts add `src` to `sys.path` using relative logic, which is fragile.
- Path issues will continue to arise unless a project-root-based approach is adopted.

---

## Sub-Tasks for Task 1

### 1A. Implement Project Root Detection Utility

- Create a utility function (e.g., `get_project_root()`) that reliably finds the project root from any script location.
- The function should look for a marker file (e.g., `.git`, `README.md`, or a custom marker) and return the absolute path to the project root.
- **VERIFICATION**: Test the utility function from multiple script locations (e.g., `scripts/verification/`, `scripts/visualization/`, `src/`) to ensure it correctly identifies the project root in all cases.
- **VERIFICATION**: Verify the function handles edge cases (e.g., when run from outside the project directory, when marker files are missing).

### 1B. Refactor Scripts to Use Project Root for File Access

- Update all scripts to import and use the project root utility.
- Replace all fragile relative path logic with paths constructed from the project root (e.g., `os.path.join(get_project_root(), 'data', 'labels', ...)`).
- Ensure all subprocess calls and sys.path modifications use the project root.
- **VERIFICATION**: Test each refactored script to ensure it works correctly from its current location and from other locations within the project.
- **VERIFICATION**: Verify that all file accesses (data loading, model loading, subprocess calls) work as expected after the refactor.

### 1C. Add Documentation and Usage Examples

- Document the new approach in the project README and/or a dedicated section in `docs/`.
- Provide code snippets showing how to use the project root utility for robust file access.
- **VERIFICATION**: Test the documented examples to ensure they work correctly and produce the expected results.
- **VERIFICATION**: Verify that the documentation is clear and complete enough for future developers to understand and use the new approach.

---

# Task 2: Improve Model Performance Evaluation Workflow

## Issue: Manual and Fragmented Model Testing Process

The current model evaluation workflow is manual, time-consuming, and lacks systematic organization. Testing different model configurations requires running individual commands, manually tracking results, and comparing performance across different parameter combinations. There's no centralized way to manage test configurations, filter results by date or model type, or systematically compare models against each other. This makes it difficult to identify the best performing models and parameters, track performance over time, or maintain a clean testing history.

## Best Solution: Implement Comprehensive Model Evaluation Framework

The most effective approach is to create a centralized model evaluation framework that includes:

- **Model Registry**: A structured database of all trained models with metadata (training date, model type, performance metrics)
- **Test Configuration Management**: Preset and customizable test configurations for different evaluation scenarios
- **Automated Testing Pipeline**: Batch testing capabilities with parallel execution and result tracking
- **Result Analysis Tools**: Interactive dashboards and comparison tools for analyzing performance
- **Data Management**: Filtering, archiving, and cleanup tools for managing test results over time

This framework will enable systematic model comparison, parameter optimization, and performance tracking while maintaining a clean and organized testing history.

## Context: Current Tools and Recent Work

### Existing Tools:

- **`scripts/analysis/performance_analysis.py`**: Analyzes sliding window performance from `test_results_analysis.csv`, generating enhanced metrics like F1 scores, precision, recall, and false positive rates per minute
- **`scripts/analysis/generate_test_results.py`**: Runs model event extraction and event verification for a single model configuration, adding results to a CSV file
- **`scripts/analysis/batch_test_models.py`**: Tests multiple model configurations with different parameters (window size, overlap, threshold)
- **`scripts/model_event_extraction/model_event_extraction.py`**: Performs sliding window event detection on preprocessed data using trained models
- **`scripts/verification/count_correctly_found_events.py`**: Compares predicted events with labeled events to calculate accuracy metrics

### Recent Accomplishments:

- Successfully moved analysis files from project root to `scripts/analysis/` directory
- Fixed path issues in all analysis scripts to work with the new directory structure
- Verified that the complete pipeline works: model event extraction → event verification → performance analysis
- Identified that the sliding window effect (many false positives from dense window sampling) is a major factor in performance, not just overfitting
- Established that the best performing configuration achieves F1=0.688, Precision=0.645, Recall=0.736 with 18.9 false positives per minute

### Current Limitations:

- No systematic way to compare models against each other
- Test results accumulate without filtering or archiving capabilities
- No standardized test configurations for different evaluation scenarios
- Manual process for identifying and testing new models
- No performance tracking over time or regression detection
- Limited visualization and analysis tools for understanding parameter sensitivity

---

## Sub-Tasks for Task 2

### 2A. Implement Model Registry and Metadata Management

- Create a model registry system (`scripts/analysis/model_registry.py`) that maintains metadata for all trained models including path, type, training date, description, and segment configuration
- Implement functions to automatically discover new models in the `models/saved_models/` directory and add them to the registry
- Create utilities to query and filter models by type, date range, or performance criteria
- **VERIFICATION**: Test model discovery with existing models in the project, verify metadata extraction works correctly
- **VERIFICATION**: Test filtering and querying functions with various criteria to ensure they return expected results
- **VERIFICATION**: Verify the registry handles edge cases (missing model files, corrupted metadata, duplicate entries)

### 2B. Create Test Configuration Management System

- Implement test configuration presets (`scripts/analysis/test_configs.py`) for common evaluation scenarios (high precision, high recall, balanced)
- Create parameter grid generation utilities that can combine preset configurations with custom parameters
- Implement configuration validation to ensure parameters are within valid ranges
- **VERIFICATION**: Test preset configurations with existing models to ensure they produce valid parameter combinations
- **VERIFICATION**: Verify parameter validation catches invalid configurations (e.g., overlap >= 1, threshold > 1)
- **VERIFICATION**: Test custom parameter overrides work correctly with preset configurations

### 2C. Enhance Batch Testing with Advanced Features

- Extend `batch_test_models.py` to support model registry integration, test configuration presets, and parallel execution
- Add progress tracking and resume capabilities for long-running batch tests
- Implement result filtering to exclude old or irrelevant test results based on date or performance criteria
- **VERIFICATION**: Test enhanced batch testing with multiple models and configurations to ensure all features work correctly
- **VERIFICATION**: Verify parallel execution improves performance without causing resource conflicts
- **VERIFICATION**: Test resume functionality by interrupting and restarting batch tests
- **VERIFICATION**: Verify result filtering correctly identifies and excludes old/invalid results

### 2D. Implement Model Comparison and Analysis Tools

- Create model comparison utilities (`scripts/analysis/model_comparison.py`) that can compare multiple models side-by-side
- Implement performance trend analysis to track how model performance changes over time
- Add parameter sensitivity analysis to understand how performance varies with different parameters
- **VERIFICATION**: Test model comparison with existing models to ensure accurate side-by-side analysis
- **VERIFICATION**: Verify trend analysis correctly identifies performance improvements or regressions over time
- **VERIFICATION**: Test parameter sensitivity analysis with known parameter ranges to ensure it produces meaningful insights

### 2E. Create Interactive Dashboard and Visualization Tools

- Implement an interactive dashboard (`scripts/analysis/dashboard.py`) using plotly or streamlit for visualizing test results
- Add filtering capabilities by model type, date range, and performance metrics
- Create performance visualization tools for comparing models and understanding parameter effects
- **VERIFICATION**: Test dashboard with existing test results to ensure all visualizations render correctly
- **VERIFICATION**: Verify filtering capabilities work as expected and update visualizations appropriately
- **VERIFICATION**: Test dashboard handles edge cases (no data, large datasets, missing model information)

### 2F. Add Automated Testing and Performance Monitoring

- Implement automated testing scheduler that monitors for new models and runs standard evaluation suites
- Create performance regression detection that alerts when model performance degrades significantly
- Add result archiving and cleanup utilities to manage test result storage over time
- **VERIFICATION**: Test automated testing with new model files to ensure they trigger appropriate evaluations
- **VERIFICATION**: Verify regression detection correctly identifies performance changes and generates appropriate alerts
- **VERIFICATION**: Test archiving and cleanup utilities to ensure they preserve important results while removing outdated data

---
