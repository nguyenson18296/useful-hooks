# useful hooks
Useful React Hooks Collection

### websites:
- https://usehooks.com/ 
- https://github.com/streamich/react-use


# Hookd

### useFormatDate

```javascript
import { useCallback } from 'react';
import { FormatDateOptions, useIntl } from 'react-intl';

type PredefinedOption =
  | 'shortDate'
  | 'shortYearMonth'
  | 'shortDateTime'
  | 'longDateShortTime'
  | '23htime'
  | 'shortDateWithoutYear';

const predefinedOptions: Record<PredefinedOption, FormatDateOptions> = {
  shortDate: { month: 'short', day: 'numeric', year: 'numeric' },
  shortDateWithoutYear: { month: 'short', day: 'numeric' },
  shortYearMonth: { month: 'short', year: 'numeric' },
  shortDateTime: { dateStyle: 'medium', timeStyle: 'short' },
  longDateShortTime: { dateStyle: 'long', timeStyle: 'short' },
  '23htime': { hour: 'numeric', minute: 'numeric', hourCycle: 'h23' },
};

/**
 * Either Date object, ISO String, or UNIX timestamp of a moment.
 */
type FormatDateValue = string | number | Date | undefined;

interface FormatDate {
  (value: FormatDateValue, options?: FormatDateOptions): string;
  (value: FormatDateValue, predefinedOption: PredefinedOption): string;
  (value: FormatDateValue, predefinedOption: PredefinedOption, additionalOptions?: FormatDateOptions): string;
}

/** Date object or UNIX timestamp */
type FormatDateTimeRangeMoment = number | Date;

interface FormatDateTimeRange {
  (from: FormatDateTimeRangeMoment, to: FormatDateTimeRangeMoment, opts?: FormatDateOptions): string;
  (from: FormatDateTimeRangeMoment, to: FormatDateTimeRangeMoment, predefinedOption: PredefinedOption): string;
  (
    from: FormatDateTimeRangeMoment,
    to: FormatDateTimeRangeMoment,
    predefinedOption: PredefinedOption,
    additionalOptions?: FormatDateOptions,
  ): string;
}

export const useFormatDate = () => {
  const intl = useIntl();

  const formatDate: FormatDate = useCallback(
    (...args) => {
      const [value, options, additionalOptions] = args;

      if (typeof options === 'object' || typeof options === 'undefined') {
        return intl.formatDate(value, options);
      }

      return intl.formatDate(value, { ...additionalOptions, ...predefinedOptions[options] });
    },
    [intl],
  );

  return formatDate;
};

export const useFormatDateTimeRange = () => {
  const intl = useIntl();

  const formatDateTimeRange: FormatDateTimeRange = useCallback(
    (...args) => {
      const [from, to, options, additionalOptions] = args;

      if (typeof options === 'object' || typeof options === 'undefined') {
        return intl.formatDateTimeRange(from, to, options);
      }

      return intl.formatDateTimeRange(from, to, { ...additionalOptions, ...predefinedOptions[options] });
    },
    [intl],
  );

  return formatDateTimeRange;
};
```

### useAsync

```javascript
import { DependencyList, useState, useCallback, useRef } from 'react';

type State<T> =
  | { status: 'init'; error?: undefined; data?: undefined }
  | { status: 'loading'; error?: any; data?: T }
  | { status: 'error'; error: any; data?: undefined }
  | { status: 'success'; error?: undefined; data: T };

type AsyncFn<Result = any, Args extends any[] = any[]> = [State<Result>, (...args: Args) => Promise<Result>];

/**
 * @returns [{ status: 'init' | 'loading' | 'error' | 'success', error, data }, callback]
 */
export const useAsync = <Result = any, Args extends any[] = any[]>(
  fn: (...args: Args) => Promise<Result>,
  deps: DependencyList,
  initialState: State<Result> = { status: 'init' },
): AsyncFn<Result, Args> => {
  const lastCallId = useRef(0);
  const [state, set] = useState<State<Result>>(initialState);

  const callback = useCallback((...args: Args) => {
    const callId = ++lastCallId.current;
    set((state) => ({ ...state, status: 'loading' }));

    return fn(...args).then(
      (data) => {
        callId === lastCallId.current && set({ data, status: 'success' });
        return data;
      },
      (error) => {
        callId === lastCallId.current && set({ error, status: 'error' });
        return error;
      },
    );
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, deps);

  return [state, callback];
};
```

### useWindowResize
```javascript
// (C) 2019-2020 GoodData Corporation
import { useEffect, useRef } from "react";
import { debounce } from "lodash";

/**
 * Hook that will call callback when browser window gets resized
 * @param callback called once window gets resized
 * @param debounceTime If specified it will debounce calls. If is 0 it will not debounce and calls will also not be deferred
 */
export const useWindowResize = (callback: () => void, debounceTime = 0) => {
    const debouncedCallback = useRef(null);
    useEffect(() => {
        debouncedCallback.current = debounceTime > 0 ? debounce(callback, debounceTime) : callback;

        window.addEventListener("resize", debouncedCallback.current);

        return () => window.removeEventListener("resize", debouncedCallback.current);
    }, [callback, debounceTime, debouncedCallback]);
};
```
