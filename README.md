# Functions



import pandas as pd

def classify_road_type(df, window_size=10, brake_threshold=0.6):
    """
    Classify each row as 'Highway' or 'City' based on speed and brake switch pattern.
    
    Parameters:
    - df: DataFrame with columns ['VIN', 'TripID', 'Timestamp', 'Speed', 'BrakeSwitch']
    - window_size: Number of previous rows to consider for brake ratio
    - brake_threshold: Threshold ratio of BrakeSwitch==1 to be considered as Highway
    
    Returns:
    - DataFrame with new 'RoadType' column
    """
    df = df.sort_values(by=['VIN', 'TripID', 'Timestamp'])  # sort by trip time
    df['RoadType'] = 'City'  # default label
    
    # Loop through each VIN & Trip group
    for (vin, trip), group in df.groupby(['VIN', 'TripID']):
        indices = group.index.tolist()
        for i in range(window_size, len(group)):
            window = group.iloc[i-window_size:i]
            current_row = group.iloc[i]
            
            brake_ratio = window['BrakeSwitch'].mean()  # ratio of BrakeSwitch == 1
            speed = current_row['Speed']
            
            if speed > 60 and brake_ratio > brake_threshold:
                df.at[indices[i], 'RoadType'] = 'Highway'
    
    return df
