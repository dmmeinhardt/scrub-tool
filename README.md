import pandas as pd
import tkinter as tk
from tkinter import filedialog
from tkinter import messagebox
from datetime import datetime
import os  # For file path manipulation

def deduplicate_csv():
    # Initialize a Tkinter root window (it won't be displayed)
    root = tk.Tk()
    root.withdraw()  # Hide the main window

    # Ask the user to select the "list" and "scrub" CSV files using a file dialog
    list_file_path = filedialog.askopenfilename(title="Select the 'list' CSV file")
    scrub_file_path = filedialog.askopenfilename(title="Select the 'scrub' CSV file")

    # Check if files were selected
    if not list_file_path or not scrub_file_path:
        messagebox.showinfo("Info", "File selection cancelled. Exiting.")
    else:
        # Specify the encoding used in your CSV files (e.g., 'utf-8', 'ISO-8859-1', 'cp1252')
        encoding = 'utf-8'

        # Load the selected CSV files into dataframes with the specified encoding
        list_df = pd.read_csv(list_file_path, encoding=encoding)
        scrub_df = pd.read_csv(scrub_file_path, encoding=encoding)

        # Replace "Phone cleaned" with "Phone Cleaned" in the list dataframe
        list_df = list_df.rename(columns={"Phone cleaned": "Phone Cleaned"})

        # Specify the columns for matching
        key_columns = ["Company", "Phone Cleaned", "Mailing Street"]

        # Convert the columns in both dataframes to lowercase for case-insensitive matching
        for column in key_columns:
            list_df[column] = list_df[column].str.lower()
            scrub_df[column] = scrub_df[column].str.lower()

        # Remove duplicates individually within the "list" dataframe, keeping the first occurrence
        list_df = list_df.drop_duplicates(subset=["Company"], keep="first")
        list_df = list_df.drop_duplicates(subset=["Phone Cleaned"], keep="first")
        list_df = list_df.drop_duplicates(subset=["Mailing Street"], keep="first")

        # Loop through each column for matching with "scrub" and remove matching rows
        for column in key_columns:
            list_df = list_df[~list_df[column].isin(scrub_df[column])]

        # Calculate the number of rows removed from the "list" dataframe
        duplicates_removed_from_list = len(list_df)

        messagebox.showinfo("Info", f"{duplicates_removed_from_list} duplicates removed from 'list' based on matches with 'scrub' (case-insensitive).")

        # Make a copy of the subset columns
        subset_columns_copy = list_df[key_columns].copy()

        # Convert the copied columns to lowercase for deduplication
        for column in key_columns:
            subset_columns_copy[column] = subset_columns_copy[column].str.lower()

        # Remove duplicates in the copied columns (case-insensitive)
        # Apply deduplication to subset_columns_copy using the lowercase columns
        subset_columns_copy = subset_columns_copy.drop_duplicates(subset=key_columns, keep="first")

        # Convert the columns in list_df back to their original case-sensitive form
        for column in key_columns:
            list_df[column] = list_df[column].str.title()

        # Merge the cleaned subset columns back into the "list" dataframe
        list_df = pd.concat([list_df, subset_columns_copy], axis=1)

        # Get the directory and file name from the original "list" file path
        output_dir, output_filename = os.path.split(list_file_path)

        # Generate a timestamp for the file name
        timestamp = datetime.now().strftime("%Y%m%d%H%M%S")

        # Define the output file path using the original directory and the timestamp
        output_list_file = os.path.join(output_dir, f'deduplicated_list_{timestamp}.csv')

        # Save the deduplicated "list" dataframe to the new CSV file, keeping only the key columns
        list_df.to_csv(output_list_file, index=False)

        messagebox.showinfo("Info", f'Deduplicated "list" saved to "{output_list_file}".')

deduplicate_csv()
