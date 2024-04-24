# Rust Written Code As A Python Package

To achieve the goal of reorganizing a large dataset of DICOM files into a BIDS-compliant format using Rust, and interfacing the Rust code with Python, I followed several steps. These steps include setting up your Rust environment, writing the Rust code, interfacing Rust with Python, and finally, calling the Rust functions from Python. The coming section guide you through each step, providing code examples and configuration details.

## Step 1: Setting Up Rust Environment on Ubuntu

1. **Install Rust:**
   First, install Rust using `rustup`, which is the official Rust toolchain manager.
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
   Follow the on-screen instructions to complete the installation.

2. **Configure Your Environment:**
   After installation, configure your current shell to access Rust tools by running:
   ```bash
   source $HOME/.cargo/env
   ```

## Step 2: Writing Rust Code

1. **Create a New Rust Project:**
   ```bash
   cargo new dicom_reorganizer
   cd dicom_reorganizer
   ```

2. **Add Dependencies:**
   Edit your `Cargo.toml` to add dependencies for CSV parsing, DICOM handling, and error handling. You'll also need `pyo3` for Python integration.
   ```toml
   [dependencies]
   csv = "1.1"
   pyo3 = { version = "0.15", features = ["extension-module"] }
   rayon = "1.5"
   anyhow = "1.0"
   ```

3. **Rust Code for CSV Reading and DICOM Reorganization:**
   In `src/lib.rs`, implement the functionality to read the CSV, reorganize DICOM files, and handle concurrency explained in previous section:

   
   ```rust
    use pyo3::prelude::*;
    use pyo3::types::PyDict;
    use rayon::prelude::*;
    use csv::ReaderBuilder;
    use std::fs::{self, copy};
    use std::path::Path;
    use anyhow::Result;

    #[pyfunction]
    fn reorganize_dicoms(csv_path: String, root_name: String, columns: &PyDict, copy_files: Option<bool>) -> PyResult<()> {
        let subject_name_column = columns.get_item("subject_name").unwrap().extract::<&str>().unwrap();
        let session_date_column = columns.get_item("session_date").unwrap().extract::<&str>().unwrap();
        let dicom_type_column = columns.get_item("dicom_type").unwrap().extract::<&str>().unwrap();
        let current_path_column = columns.get_item("current_path").unwrap().extract::<&str>().unwrap();

        let should_copy = copy_files.unwrap_or(true); // Default to true if not specified

        let mut rdr = ReaderBuilder::new().from_path(csv_path).unwrap();
        let headers = rdr.headers().unwrap().clone();

        rdr.records().par_bridge().for_each(|result| {
            let record = result.unwrap();
            let subject_name = record.get(headers.iter().position(|h| h == subject_name_column).unwrap()).unwrap();
            let session_date = record.get(headers.iter().position(|h| h == session_date_column).unwrap()).unwrap();
            let dicom_type = record.get(headers.iter().position(|h| h == dicom_type_column).unwrap()).unwrap();
            let current_path = record.get(headers.iter().position(|h| h == current_path_column).unwrap()).unwrap();

            // Add 'sub-' and 'ses-' prefixes
            let new_path = format!("{}/sub-{}/ses-{}/{}/", root_name, subject_name, session_date, dicom_type);
            fs::create_dir_all(&new_path).unwrap();

            // Option to copy or move
            let destination_path = Path::new(&new_path).join(Path::new(current_path).file_name().unwrap());
            if should_copy {
                copy(current_path, &destination_path).unwrap();
            } else {
                fs::rename(current_path, destination_path).unwrap();
            }
        });

        Ok(())
    }

    #[pymodule]
    fn dicom_reorganizer(_py: Python, m: &PyModule) -> PyResult<()> {
        m.add_function(wrap_pyfunction!(reorganize_dicoms, m)?)?;
        Ok(())
    }
   ```
   This code snippet demonstrates how to read a CSV file, reorganize DICOM files concurrently, and implement error handling. Note that for brevity and simplicity, error handling here is minimal and should be expanded based on your specific requirements.

4. **Build the Rust Code:**
   To compile the Rust code as a Python extension, add the following lines to your `Cargo.toml`:
   ```toml
   [lib]
   name = "dicom_reorganizer"
   crate-type = ["cdylib"]
   ```
   Then build the project using `cargo build --release`. This will generate a shared library in `target/release` that you can import from Python.

## Step 3: Interfacing Rust with Python

1. **Install `maturin`:**
   `maturin` is a tool to build and publish Rust-based Python packages easily.
   ```bash
   pip install maturin
   ```

2. **Build the Python Package:**
   Run `maturin develop` or `maturin build --release` and then `pip install target/wheels/<wheel_name>.whl` from the project root directory to build your Rust project and make it available as a Python module.

## Step 4: Calling Rust Functions from Python

After building your Python package with `maturin`, you can import and use your Rust functions in Python as follows:

```python
import dicom_reorganizer

root_name = "path/to/project_bids_folder"
columns = {
    "subject_name": "SubjectID",
    "session_date": "AcquisitionDate",
    "dicom_type": "DicomType",
    "current_path": "DicomPath",
}

# Replace 'your_csv_file.csv' with the path to your actual CSV file
dicom_reorganizer.reorganize_dicoms('your_csv_file.csv', root_name, columns)
```

This setup allows you to leverage Rust's performance benefits, including concurrency and safe error handling, while integrating seamlessly with your Python ecosystem.

!!!info Additional Steps

    - **Testing and Validation:** Before using this setup in a production environment, thoroughly test the functionality with a subset of your dataset to ensure everything works as expected.
    - **Error Handling:** Enhance error handling in Rust and Python to manage exceptions and unexpected situations gracefully.
    - **Performance Tuning:** Depending on your dataset's size and structure, you may need to adjust the concurrency settings or optimize file handling operations.