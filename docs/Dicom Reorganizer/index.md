# Dicom Reorganizer

**Overview**

The provided code is a Rust library that is designed to work with Python using the `pyo3` crate, which allows Rust code to be called from Python. The purpose of this code is to reorganize DICOM files based on information provided in a CSV file.

**Requirements:** [[Source]](https://github.com/NeuroGranberg/dicom_reorganizer/blob/ee470fe40bc3c59bcce9b9cca48afcf73b334370/src/lib.rs#L1C1-L7C20)

1. The code requires the following Rust crates:
    - `pyo3`: Enables integration with Python.
    - `rayon`: Provides parallelism for faster processing.
    - `csv`: Allows reading CSV files.
    - `anyhow`: Helps with error handling.

        ```rust
        use pyo3::prelude::*;
        use pyo3::types::PyDict;
        use rayon::prelude::*;
        use csv::ReaderBuilder;
        use std::fs::{self, copy};
        use std::path::Path;
        use anyhow::Result;
        ```

2. It expects a CSV file containing the following columns:
    - `subject_name`: Represents the subject or patient name.
    - `session_date`: Represents the date of the session.
    - `dicom_type`: Represents the type of DICOM file (e.g., anat, func, etc).
    - `current_path`: Represents the current file path of the DICOM file.


!!!info Creating csv of your dicom data

    I build a tool to extract all tags (alongside path) of every disom in a given dataset that can be used to build the needed csv file for this step. You can find more information if you look at [Dicom Metadata Extractor](https://neurogranberg.github.io/Nima_Documentation/Dicom%20Extractor/Usage/)

**Functionality:** [[Source]](https://github.com/NeuroGranberg/dicom_reorganizer/blob/fe6f2164c39ee490a68cdec89d416673491a18cd/src/lib.rs#L9C1-L42C2)

The code defines a Python function `reorganize_dicoms` that takes the following arguments:

1. `csv_path` (String): The path to the CSV file containing the DICOM file information.
2. `root_name` (String): The root directory name where the reorganized files will be stored.
3. `columns` (PyDict): A Python dictionary containing the column names for `subject_name`, `session_date`, `dicom_type`, and `current_path`.
4. `copy_files` (Option<bool>): An optional boolean flag indicating whether to copy or move the files. If not provided, it defaults to `true` (copy).



    ```rust
    #[pyfunction]
    fn reorganize_dicoms(
        csv_path: String,
        root_name: String,
        columns: &PyDict,
        copy_files: Option<bool>
        ) -> PyResult<()> {
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
    ```

The function performs the following steps:

1. Reads the CSV file using the `csv` crate.
2. Iterates over each record (row) in the CSV file in parallel using `rayon`.
3. For each record:
    - Extracts the values for `subject_name`, `session_date`, `dicom_type`, and `current_path`.
    - Constructs a new path following the BIDS format: `{root_name}/sub-{subject_name}/ses-{session_date}/{dicom_type}/`.
    - Creates the necessary directories for the new path.
    - Determines whether to copy or move the file based on the `copy_files` flag.
    - Copies or moves the DICOM file to the new path, preserving the original file name.

**Integration with Python:**  [[Source]](https://github.com/NeuroGranberg/dicom_reorganizer/blob/fe6f2164c39ee490a68cdec89d416673491a18cd/src/lib.rs#L44C1-L48C2)

The `#[pymodule]` section defines a Rust module named `dicom_reorganizer` that can be imported and used in Python. It adds the `reorganize_dicoms` function to the module, allowing it to be called from Python.

```rust
#[pymodule]
fn dicom_reorganizer(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(reorganize_dicoms, m)?)?;
    Ok(())
}
```

To use this code in Python, you would need to:

1. Compile the Rust code into a Python package or library.
2. Import the `dicom_reorganizer` module in your Python code.
3. Call the `reorganize_dicoms` function, providing the required arguments.

These stepd explained throughly in the next section.