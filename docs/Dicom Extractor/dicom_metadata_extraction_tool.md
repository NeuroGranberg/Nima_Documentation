# Extraction Tool

This Python script, `DICOMetaExtractor_v32.py`, is designed to efficiently extract metadata from DICOM files across a directory structure, leveraging advanced data processing libraries to handle large datasets effectively. The tool outputs the extracted metadata into a CSV file, providing a comprehensive overview of the DICOM files processed.

## Prerequisites

- Python 3.7 or newer
- pip (Python package installer)

## Installation

Before running the script, ensure you have the required libraries installed. The primary libraries used in this version are `pydicom`, `polars`, `tqdm`, `pandas`, and `portalocker`. You can install these libraries using pip:

```sh
pip install pydicom polars tqdm pandas portalocker
```

Ensure all dependencies are installed successfully before proceeding to use the script.

## Biuld

Here's a breakdown of the code, its functionality, and potential points to consider:

**Overall Purpose**

The code is designed to extract metadata from DICOM files (a standard medical image format) and export the extracted data into a CSV file. Key features include:

* **Multiprocessing and Threading:** It leverages parallel processing to efficiently handle multiple DICOM folders and files.
* **Data Cleaning:** It handles missing values and ensures consistent data representation within the CSV.
* **Scalability:** It's able to process large collections of DICOM files by breaking the work down into smaller chunks for processing.

### Code Breakdown

### Import Statements

* `concurrent.futures`: Provides tools for running tasks concurrently using threads or processes.
* `tqdm`: Displays progress bars during operations.
* `polars` and `pandas`: Core data analysis libraries.
* `uuid`: Generates unique identifiers.
* `os`: For file system interactions.
* `sys`: Access to system-level functions.
* `argparse`: For command-line argument parsing.
* `pydicom`: Library for handling DICOM files.
* `json`: For loading and saving JSON data.
* `shutil`: For file and directory operations.
* `warnings`: To suppress warnings from the `pydicom` library.
* `portalocker`: Manages file locking to prevent simultaneous updates.

### Functions

1. **`dicom_tag_to_dict(dicom_file_path)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L18C1-L48C28)
    ```python
    def dicom_tag_to_dict(dicom_file_path):
        dcm_dict = pydicom._dicom_dict.DicomDictionary
        dicom_file = pydicom.dcmread(
            dicom_file_path, specific_tags=list(dcm_dict.keys())[:-14]
        )
        results_tag_dict = {}

        def process_element(element, prefix=""):
            if isinstance(element, pydicom.DataElement) and not isinstance(
                element.value, pydicom.sequence.Sequence
            ):
                if not (
                    element.tag.group >> 8 == 0x60 and (element.tag.group & 0xFF) % 2 == 0
                ) and not element.tag.group in (0x7FE0, 0xFFFA, 0xFFFC, 0xFFFE):
                    tag_key = f"{prefix}{element.name} {element.tag}"
                    results_tag_dict[tag_key] = element.value
            elif isinstance(element, pydicom.Dataset) or isinstance(
                element.value, pydicom.sequence.Sequence
            ):
                prefix = (
                    f"{prefix}{element.name} {element.tag} - "
                    if isinstance(element, pydicom.DataElement)
                    else prefix
                )
                for sub_element in (
                    element if isinstance(element, pydicom.Dataset) else element.value
                ):
                    process_element(sub_element, prefix)

        process_element(dicom_file)
        return results_tag_dict
    ```
    * Loads a DICOM file with `pydicom`.
    * Extracts specific metadata tags, avoiding overlay and private tags for cleaner results.
    * Recursively processes elements within the DICOM data structure, storing extracted tags and values in a dictionary.

2. **`stringify_value(value)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L51C1-L64C38)

    ```python
    def stringify_value(value):
        if value is None:
            return "N/A"
        elif isinstance(value, bytes):
            return value.hex()
        else:
            try:
                return (
                    json.dumps(value)
                    if isinstance(value, (list, dict, set))
                    else str(value)
                )
            except Exception as e:
                return f"Error: {str(e)}"
    ```

    * Converts different data types into string representations suitable for CSV output.
    * Serializes lists, dictionaries, and sets using JSON.
    * Handles errors that might occur during conversion.

3. **`process_file(filepath)`**  [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L67C1-L76C39)

    ```python
    def process_file(filepath):
        try:
            dicom_data = dicom_tag_to_dict(filepath)
            for key, value in dicom_data.items():
                dicom_data[key] = stringify_value(value)
            dicom_data["DicomPath"] = filepath
            return dicom_data
        except Exception as e:
            print(f"Error processing file {filepath}: {str(e)}")
            return {"DicomPath": filepath}
    ```

    * Calls `dicom_tag_to_dict` to extract metadata from a DICOM file.
    * Calls `stringify_value` to format the extracted values.
    * Adds the original file path to the data.
    * Includes error handling.

4. **`process_folder(folder)`**  [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L78C1-L94C23)

    ```python
    def process_folder(folder):
        filepaths = [
            os.path.join(folder, f) for f in os.listdir(folder) if f.endswith(".dcm")
        ]
        chunk_size = 6
        chunks = [
            filepaths[i : i + chunk_size] for i in range(0, len(filepaths), chunk_size)
        ]
        all_results = []
        with ProcessPoolExecutor(max_workers=6) as executor:
            future_to_chunk = {
                executor.submit(process_files_chunk, chunk): chunk for chunk in chunks
            }
            for future in as_completed(future_to_chunk):
                all_results.extend(future.result())

        return all_results
    ```

    * Gets a list of DICOM files in a folder.
    * Splits files into chunks for parallel processing.
    * Uses `ProcessPoolExecutor` to distribute `process_files_chunk` across multiple processes.

5. **`process_files_chunk(filepaths)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L97C1-L98C62)

    ```python
    def process_files_chunk(filepaths):
        return [process_file(filepath) for filepath in filepaths]
    ```

    * Calls `process_file` for each DICOM file within the chunk.

6. **`create_temp_dir(output_path)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L101C1-L106C25)

    ```python
    def create_temp_dir(output_path):
        base_dir = os.path.dirname(output_path)
        temp_dir_name = "temp_processing"
        temp_dir_path = os.path.join(base_dir, temp_dir_name)
        os.makedirs(temp_dir_path, exist_ok=True)
        return temp_dir_path
    ```

    * Creates a temporary directory to store intermediate results.

7. **`save_partial_results(folder_results, temp_dir, index)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L109C1-L113C70)

    ```python
    def save_partial_results(folder_results, temp_dir, index):
        unique_id = uuid.uuid4()
        partial_output = os.path.join(temp_dir, f"part_{index}_{unique_id}.json")
        with open(partial_output, "w", encoding="utf-8") as file:
            json.dump(folder_results, file, ensure_ascii=False, indent=4)
    ```

    * Saves a chunk of processed results as a JSON file in the temporary directory.

8. **`replace_with_none(value)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L115C1-L119C21)

    ```python
    def replace_with_none(value):
        if value in ["", "N/A", "None", "NONE", None]:
            return None
        else:
            return value
    ```

    * Replaces empty values with `None` for cleaner CSV representation.

9. **`merge_partial_results_and_cleanup(temp_dir, output_path)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L121C1-L148C28)

    ```python
    def merge_partial_results_and_cleanup(temp_dir, output_path):
        print(
            f"\nMergging all partial results into {output_path} and cleaning up temporary files..."
        )
        partial_files = [
            os.path.join(temp_dir, f) for f in os.listdir(temp_dir) if f.endswith(".json")
        ]
        if not partial_files:
            print("No DICOM files were processed. No CSV file will be generated.")
            shutil.rmtree(temp_dir)
            return

        all_data = []
        for json_file in partial_files:
            try:
                with open(json_file, "r", encoding="utf-8") as file:
                    data = json.load(file)
                    all_data.extend(data)
            except Exception as e:
                print(f"Failed to load {json_file}: {e}")

        final_df = pl.from_pandas(pd.DataFrame(all_data))
        final_df = final_df.with_columns(pl.all().map_elements(replace_with_none))
        final_df = final_df[
            [s.name for s in final_df if not (s.null_count() == final_df.height)]
        ]
        final_df.write_csv(output_path)
        shutil.rmtree(temp_dir)
    ```

    * Loads partial results from JSON files.
    * Converts data into a Polars dataframe, handling missing values.
    * Saves the final dataframe as a CSV file.
    * Deletes the temporary directory

10. **`load_processed_folders(base_dir)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L151C1-L161C21)

    ```python
    def load_processed_folders(base_dir):
        try:
            with open(f"{base_dir}/processed_folders.json", "r") as file:
                portalocker.lock(file, portalocker.LOCK_SH)
                data = json.load(file)
                portalocker.unlock(file)
                return set(data)
        except FileNotFoundError:
            return set()
        except json.decoder.JSONDecodeError:
            return set()
    ```

    * Loads a list of previously processed folders to avoid redundant work.

11. **`mark_folder_as_processed(folder, output_path)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L164C1-L171C33)

    ```python
    def mark_folder_as_processed(folder, output_path):
        base_dir = os.path.dirname(output_path)
        processed = load_processed_folders(base_dir)
        processed.add(folder)
        with open(f"{base_dir}/processed_folders.json", "w") as file:
            portalocker.lock(file, portalocker.LOCK_EX)
            json.dump(list(processed), file)
            portalocker.unlock(file)
    ```

    * Updates tracking of processed folders to prevent reprocessing.

12. **`check_dcm_files(directory)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L173C1-L178C16)

    ```python
    def check_dcm_files(directory):
        """Check if a directory contains any .dcm files."""
        for subdir, _, files in os.walk(directory):
            if any(fname.endswith(".dcm") for fname in files):
                return subdir
        return None
    ```
    * It checks if a directory contains any `.dcm` files.

13. **`find_dcm_folders(root_dir)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L180C1-L195C23)

    ```python
    def find_dcm_folders(root_dir):
        with ThreadPoolExecutor(max_workers=4) as executor:
            futures = {
                executor.submit(check_dcm_files, os.path.join(root_dir, subdir)): subdir
                for subdir, _, _ in os.walk(root_dir)
            }
            dcm_folders = []
            for future in tqdm(
                as_completed(futures), total=len(futures), desc="Checking directories"
            ):
                result = future.result()
                if result:
                    dcm_folders.append(result)
        dcm_folders = list(set(dcm_folders))
        dcm_folders.sort()
        return dcm_folders
    ```

    * Locates subfolders containing DICOM files using a `ThreadPoolExecutor`.

14. **`process_folder_and_save_results(...)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L198C1-L202C54)

    ```python
    def process_folder_and_save_results(folder, temp_dir, processed_folders, output_path):
        if folder not in processed_folders:
            folder_results = process_folder(folder)
            save_partial_results(folder_results, temp_dir, len(os.listdir(temp_dir)))
            mark_folder_as_processed(folder, output_path)
    ```

    * Processes a folder if it hasn't been processed before and saves partial results

15. **`collect_and_process_dicom_data(...)`** [[Source]](https://github.com/NeuroGranberg/DICOMetaExtractor/blob/8a594f32b0efb1b199ff00e8b1082f5714e861cd/DICOMetaExtractor_v32.py#L205C1-L225C61)

    ```python
    def collect_and_process_dicom_data(root_dir, output_path):
        base_dir = os.path.dirname(output_path)
        temp_dir = create_temp_dir(output_path)
        dcm_folders = find_dcm_folders(root_dir)
        processed_folders = load_processed_folders(base_dir)
        with ProcessPoolExecutor(max_workers=12) as executor:
            futures = [
                executor.submit(
                    process_folder_and_save_results,
                    folder,
                    temp_dir,
                    processed_folders,
                    output_path,
                )
                for folder in dcm_folders
            ]
            for future in tqdm(
                as_completed(futures), total=len(futures), desc="Processing folders"
            ):
                folder_processed = future.result()
        merge_partial_results_and_cleanup(temp_dir, output_path)
    ```

    * Core orchestration: Finds folders, processes them in parallel, merges final results.
