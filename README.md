-- Custom function to calculate the number of days in a specified month of a specific year
CREATE OR REPLACE FUNCTION calculate_days(mnth NUM, yr NUM) RETURN NUM IS
    total_days_in_month NUM;  -- Variable to hold the total days
BEGIN
    -- Determine the last day of the month and convert to a number
    total_days_in_month := TO_NUMBER(TO_CHAR(LAST_DAY(TO_DATE(yr || '-' || mnth || '-01', 'YYYY-MM-DD')), 'DD'));
    RETURN total_days_in_month;  -- Output the total days
END;

-- Procedure to evaluate attendance for employees for a particular month and year
CREATE OR REPLACE PROCEDURE evaluate_attendance(mnth NUM, yr NUM) IS
    pres_count NUM := 0;  -- Count of days present
    abs_count NUM := 0;   -- Count of days absent
    total_days NUM;      -- Total days in the month
    emp_full_name VARCHAR2(100);  -- Full name of the employee
BEGIN
    -- Get the total number of days in the specified month
    total_days := calculate_days(mnth, yr);

    -- Loop through each employee to gather attendance data
    FOR emp IN (SELECT employee_id, first_name || ' ' || last_name AS name FROM employees) LOOP
        pres_count := 0;  -- Reset present count for new employee
        abs_count := 0;   -- Reset absent count for new employee

        -- Check the attendance records for the employee
        FOR record IN (SELECT status FROM attendance
                       WHERE employee_id = emp.employee_id
                       AND EXTRACT(MONTH FROM attendance_date) = mnth
                       AND EXTRACT(YEAR FROM attendance_date) = yr) LOOP
            -- Count the present and absent days
            IF record.status = 'Present' THEN
                pres_count := pres_count + 1;  -- Increment for present
            ELSE
                abs_count := abs_count + 1;    -- Increment for absent
            END IF;
        END LOOP;

        -- Display the attendance information
        DBMS_OUTPUT.PUT_LINE('Employee: ' || emp.name || ' - Days Present: ' || pres_count || ', Days Absent: ' || abs_count);

        -- Calculate and display the attendance percentage
        IF total_days > 0 THEN
            DBMS_OUTPUT.PUT_LINE('Attendance Rate: ' || ROUND((pres_count / total_days) * 100, 2) || '%');
        ELSE
            DBMS_OUTPUT.PUT_LINE('No attendance data for ' || emp.name || ' this month.');
        END IF;
    END LOOP;
END;
