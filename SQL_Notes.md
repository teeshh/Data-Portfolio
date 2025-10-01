📝 Topic 1: Using CASE + Aggregates (SQL Pivot Trick)
    
    Problem:
    We often want to turn rows into columns.
    Example: For each (machine_id, process_id) you have 2 rows (start/end), but you want them side by side.
    
    Input table:
    machine_id	process_id	activity_type	  timestamp
        0            0	        start	        0.712
        0	           0	         end	        1.520
    
    Goal output:
    machine_id	process_id	start_ts	end_ts
        0         	0	        0.712	   1.520

    Solution - (Pivot Trick)
        SELECT
          machine_id,
          process_id,
          MAX(CASE WHEN activity_type = 'start' THEN timestamp END) AS start_ts,
          MAX(CASE WHEN activity_type = 'end'   THEN timestamp END) AS end_ts
        FROM Activity
        GROUP BY machine_id, process_id;
    
    How it works:
    CASE turns rows into either the value we want or NULL.
    Since only one row matches per group, there’s only one non-NULL.
    MAX (or MIN) extracts that value.
    GROUP BY ensures we compute per (machine_id, process_id).
    
    Example breakdown:
    Group (0,0) has two rows:
    start row → CASE start = 0.712
    end row → CASE end = 1.520
    
    So:
    start_ts = MAX(0.712, NULL) = 0.712
    end_ts = MAX(NULL, 1.520) = 1.520
    
    ✅ Shortcut takeaway
    Use MAX(CASE WHEN condition THEN value END) in a grouped query to pivot rows into columns.
    Efficient: avoids self-joins, only one scan of the table.

📝 Topic 2: Comparing Sequential Rows in SQL
    
    Problem:
    You need to compare a row with the previous or next row (e.g., today vs yesterday’s temperature).
    
    Input table:
    id	recordDate	temperature
    1	  2020-01-01	    10
    2	  2020-01-02	    12
    3	  2020-01-03	    8
    
    Goal: Find days where today’s temperature > yesterday’s.
    
    Solution - (Self-Join with Shift)
      SELECT today.id, today.recordDate, today.temperature, yesterday.temperature
      FROM Weather today
      JOIN Weather yesterday
        ON today.recordDate = DATE_ADD(yesterday.recordDate, INTERVAL 1 DAY)
      WHERE today.temperature > yesterday.temperature;
    
    How it works:
    Align today’s row with yesterday’s row by shifting the date.
    Now you can directly compare today.temp vs yesterday.temp.
    
    Generalized Pattern:
    JOIN table alias2
      ON alias1.sequence_col = alias2.sequence_col + (shift amount)
    sequence_col: the column that defines order (date, id, etc).
    shift amount: how far forward/back you compare (1 day, 1 id, etc).
    
    ✅ Shortcut takeaway
    Use a self-join with a shifted condition to compare sequential rows.
    E.g., today vs yesterday, current vs last month, current vs previous id.
