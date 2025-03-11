import pandas as pd
from datetime import datetime, timedelta

def fill_data_gap(csv_file_path, date_column, value_column, oneview, gap_month):
   
    # Read the CSV file into a DataFrame
    df = pd.read_csv(csv_file_path)
    
    # Convert the date column to datetime
    df[date_column] = pd.to_datetime(df[date_column])
    
    # Filter by oneview
    df_filtered = df[df['oneview'] == oneview]
    
    # Check if any data exists for this oneview
    if df_filtered.empty:
        return None, "No data", f"No data found for oneview {oneview}"
    
    # Convert gap_month to datetime if it's a string
    if isinstance(gap_month, str):
        gap_month = pd.to_datetime(gap_month)
    
    # BUSINESS RULE - STEP 1 (TYPE 1)
    # Calculate the date three months before the gap
    three_months_before_gap = gap_month - timedelta(days=90)
    
    # Get data for the 3 months before the gap
    previous_three_months = df_filtered[
        (df_filtered[date_column] >= three_months_before_gap) & 
        (df_filtered[date_column] < gap_month)
    ]
    
    # Group by month to check for consecutive months
    monthly_data = previous_three_months.groupby(previous_three_months[date_column].dt.to_period('M')).agg({
        value_column: 'sum',
        date_column: 'count'
    })
    
    # Check if we have data for each of the three preceding months
    if len(monthly_data) == 3:
        # Calculate the average of the three months
        average_value = monthly_data[value_column].mean()
        months_covered = [period.strftime('%Y-%m') for period in monthly_data.index.to_timestamp()]
        return average_value, "Type 1", f"Three-month average ({', '.join(months_covered)})"
    
    # BUSINESS RULE - STEP 2 (TYPE 2)
    # If three consecutive months data not available, check previous year's same month
    previous_year = gap_month.year - 1
    same_month_previous_year = df_filtered[
        (df_filtered[date_column].dt.year == previous_year) & 
        (df_filtered[date_column].dt.month == gap_month.month)
    ]
    
    if not same_month_previous_year.empty:
        previous_year_value = same_month_previous_year[value_column].sum()
        previous_period = f"{previous_year}-{gap_month.month:02d}"
        return previous_year_value, "Type 2", f"Previous year's value ({previous_period})"
    
    # If neither rule can be applied
    return None, "No suitable data", "Could not apply either business rule to fill the gap"

# Example usage
csv_file_path = 'C:\\ESG\\esgRebuild\\electricity.csv'
date_column = 'period'
value_column = 'consumption'
oneview_number = '77399X'
gap_month = ''  # Month with missing data
monthly_data = '2025-01-01'




#value, rule_type, details = fill_data_gap
average, start_date, end_date = monthly_data(
    csv_file_path, 
    date_column, 
    value_column, 
    oneview_number, 
    gap_month,
    monthly_data,
)
print(f'Average value from {start_date.strftime("%Y-%m-%d")} to {end_date.strftime("%Y-%m-%d")}: {average}')

if value is not None:
    print(f'Average value from {start_date.strftime("%Y-%m-%d")} to {end_date.strftime("%Y-%m-%d")}: {average}')
    print(f"Gap filled successfully using {rule_type}")
    print(f"Value: {value}")
    print(f"Details: {details}")
else:
    print(f"Failed to fill gap: {details}")




