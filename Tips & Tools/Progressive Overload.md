- Much needed for a promotion/ CV Update/ Interviews
- Just start it right when a challenge arises, then document it along the way
- Focus more on the IMPACT (especially ***meaningful*** ones) rather than the output just for the sake of getting something done.
- Have the STAR method as your guidelines
- Update every month/quarter
- Make a summary of **highest-impact** checkpoint - the works that you're most proud of, after a year of working (or before switching to a new company)
- Make YOUR achievements sound like they're bringing a meaningful change for your team/company/community (i.e. if you have time, try to write a dramatic STORY)
---

### FPT
- Biggest flex: refractor code to embrace React design pattern && readability. 
	- E.g. refractor repeated code:
```jsx
// sonarlint(javascript:S1871)
// returning exact same thing for 2 conditions:
case inputType.DATE:
        if (item['key'] === 'dateOfBirth')
          return (
            <DatePicker
              className={styles['date-picker']}
              format="DD/MM/YYYY"
              onChange={(_date, stringDate) => {
                form.setFieldsValue(`${item['key']}`, stringDate);
              }}
              disabledDate={(current) => current && current > moment().endOf('day')}
            />
          );

        if (item['key'] == 'ptpTime' && showPTPTime)
          return (
            <DatePicker
              className={styles['date-picker']}
              format="H:mm DD-MM-YYYY"
              onSelect={setSelectedDate}
              onChange={(_date, stringDate) => {
                form.setFieldsValue(`${item['key']}`, stringDate);
              }}
              showTime={{
                hideDisabledOptions: true,
                disabledHours: () => {
                  if (moment().date() == selectedDate?.date()) return disHrs;
                  else return [];
                },
                disabledMinutes: () => {
                  if (
                    moment().hour() == selectedDate?.hour() &&
                    moment().date() == selectedDate?.date()
                  )
                    return disMins;
                  else return [];
                },
              }}
              disabledDate={(current) => current && current <= moment().subtract(1, 'days')}
            />
          );

        else if (item['key'] == 'recallAt' && isRecallAtChecked)
          return (
            <DatePicker
              className={styles['date-picker']}
              format="H:mm DD-MM-YYYY"
              onSelect={setSelectedDate}
              onChange={(_date, stringDate) => {
                form.setFieldsValue(`${item['key']}`, stringDate);
              }}
              showTime={{
                hideDisabledOptions: true,
                disabledHours: () => {
                  if (moment().date() == selectedDate?.date()) return disHrs;
                  else return [];
                },
                disabledMinutes: () => {
                  if (
                    moment().hour() == selectedDate?.hour() &&
                    moment().date() == selectedDate?.date()
                  )
                    return disMins;
                  else return [];
                },
              }}
              disabledDate={(current) => current && current <= moment().subtract(1, 'days')}
            />
          );

// Refractored to:
case inputType.DATE:

        if (item['key'] === 'dateOfBirth')
          return (
            <DatePicker
              className={styles['date-picker']}
              format="DD/MM/YYYY"
              onChange={(_date, stringDate) => {
                form.setFieldsValue(`${item['key']}`, stringDate);
              }}
              disabledDate={(current) => current && current > moment().endOf('day')}
            />
          );
        if (
          (item['key'] == 'ptpTime' && showPTPTime) ||
          (item['key'] == 'recallAt' && isRecallAtChecked)
        )

          // <DatePicker/> with time:
          return (
            <DatePicker
              className={styles['date-picker']}
              format="H:mm DD-MM-YYYY"
              onSelect={setSelectedDate}
              onChange={(_date, stringDate) => {
                form.setFieldsValue(`${item['key']}`, stringDate);
              }}
              showTime={{
                hideDisabledOptions: true,
                disabledHours: () => {
                  if (moment().date() == selectedDate?.date()) return disHrs;
                  else return [];
                },
                disabledMinutes: () => {
                  if (
                    moment().hour() == selectedDate?.hour() &&
                    moment().date() == selectedDate?.date()
                  )
                    return disMins;
                  else return [];
                },
              }}
              disabledDate={(current) => current && current <= moment().subtract(1, 'days')}
            />
          );
```
- Wired up APIs to work with my new UI
- Extension (new design): refractor legacy code using Antd `<ProTable/>` to use f2p `<Table/>`
- vdong 
	- remove/add some fields, set fixed first 2 rows
	- PopUp UI Customer Info: `<Radio.Group/>`, disable date/time