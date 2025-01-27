import pandas as pd

def gen_var_hh_size(df, hh_id_col='IDMEN'):
    """Generates household size and counts of dependent children and elderly.
   
    Parameters
    ----------
    df : DataFrame
        Input DataFrame containing individual data.
    hh_id_col : str, optional
        Household identifier column name, by default 'hh_id'
   
    Returns
    -------
    DataFrame
        DataFrame with household size, number of children, and number of elderly added.
    """
   
    # ======================================================
    # GENERATE HH SIZE
    # ======================================================
    # Calculate household size
    hh_size = df.groupby(hh_id_col).size().reset_index(name='hh_size')

    # ======================================================
    # GENERATE NUMBER OF CHILDREN AND ELDERLY
    # ======================================================
    # Count number of children (age <= 15)
    num_children = df[df['age'] <= 15].groupby(hh_id_col).size().reset_index(name='num_children')

    # Count number of elderly (age >= 65)
    num_elderly = df[df['age'] >= 65].groupby(hh_id_col).size().reset_index(name='num_elderly')

    # ======================================================
    # MERGE THE DATAFRAMES
    # ======================================================
    df_hh = hh_size.merge(num_children, on=hh_id_col, how='left') \
                    .merge(num_elderly, on=hh_id_col, how='left')

    # ======================================================
    # FILL NAs WITH 0
    # ======================================================
    df_hh.fillna({'num_children': 0, 'num_elderly': 0}, inplace=True)

    # ======================================================
    # CHECK THAT WE HAVE ALL HH_ID
    # ======================================================
    if df_hh[hh_id_col].nunique() != df[hh_id_col].nunique():
        print("Warning: Not all household IDs are present in the summary.")

    return df_hh

# Load the dataset into a Pandas DataFrame
df = pd.read_csv(small_dataset_path)

# ====================================
# ADD HOUSEHOLD LEVEL VARIABLES
# ====================================

# Generate household level variables
df_hh = gen_var_hh_size(df)

# Merge household size back to the main DataFrame
df = df.merge(df_hh, on='IDMEN', how='left')

# Optional: Display the first few rows of the updated DataFrame
print(df.head())
