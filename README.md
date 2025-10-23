# Sen-s-slope
def sen_slope_with_ci(ts: pd.Series, ci_lower=10, ci_upper=90):# 90 confidence by default, 5th and 95th percentiles
    """
    Compute Theil–Sen slope and confidence interval for irregular time series.

    Parameters
    ----------
    ts : pd.Series
        Series with datetime index and concentration values (µg/m³)
    ci_lower, ci_upper : float
        Confidence percentiles to return, e.g. 10 and 90 for 10–90% CI

    Returns
    -------
    slope : float
        Median Theil–Sen slope (µg/m³ per year)
    lcl : float
        Lower confidence limit (µg/m³ per year)
    ucl : float
        Upper confidence limit (µg/m³ per year)
    """

    ts = ts.dropna()
    if len(ts) < 3:
        return np.nan, np.nan, np.nan

    # Convert datetime to fractional years
    t = (ts.index - ts.index[0]).days / 365.25
    y = ts.values.astype(float)

    # Compute all pairwise slopes
    slopes = []
    n = len(y)
    for i in range(n - 1):
        dt = t[i + 1:] - t[i]
        dy = y[i + 1:] - y[i]
        slopes.extend(list(dy[dt != 0] / dt[dt != 0]))

    if not slopes:
        return np.nan, np.nan, np.nan

    slopes = np.array(slopes)
    slope = np.median(slopes)
    lcl = np.percentile(slopes, ci_lower)
    ucl = np.percentile(slopes, ci_upper)

    return slope, lcl, ucl

