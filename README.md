## Imports
# import packages and add src to system path to import functions
import pandas as pd
import os

import sys

src_path = os.path.abspath(os.path.join(os.getcwd(), "..", "src"))
if src_path not in sys.path:
    sys.path.append(src_path)

from data_utils import restrict_to_alerted_users, add_stratification
from sampling_utils import get_stratified_random_sample, write_to_new_file
from input_utils import (
    get_survey_code,
    get_regions_with_alerts_list,
    get_sampling_pool_filepath,
    get_sampling_proportion,

## Survey Inputs

# enter survey code, e.g. 2025H01
survey_code = get_survey_code()

# enter regions with alerts (or 'all' if all had alerts)
regions_with_alerts = get_regions_with_alerts_list()

# enter filename, e.g. sampling_pool-20250619
excel_filepath = get_sampling_pool_filepath()

# enter proportion to sample, normally 0.2 but may vary
proportion_to_sample = get_sampling_proportion()

## Load and transform data for sampling

# load registered users from Excel spreadsheet
users = pd.read_excel(
    excel_filepath, sheet_name="SurveySamplingPool", header=1, usecols="B:Q"
)

# restricted to alerted users
users_alerted = restrict_to_alerted_users(users, regions_with_alerts)

# split into public and professional groups
pub = users_alerted[users_alerted["SurveyGroup"] == "Public"].copy()
pro = users_alerted[users_alerted["SurveyGroup"] == "Professionals"].copy()

# add columns for stratification of public (by region) and professionals (by region and role)
pub = add_stratification(pub, regions_with_alerts, "pub")
pro = add_stratification(pro, regions_with_alerts, "pro")

## Random Sampling

# create samples
pub_sample = get_stratified_random_sample(pub, "strat_region", proportion_to_sample)
pro_sample = get_stratified_random_sample(
    pro, "strat_region_role", proportion_to_sample
)

## Write output file

# write to new sheet in output file
write_to_new_file(pub_sample, "pub", survey_code)
write_to_new_file(pro_sample, "pro", survey_code)
