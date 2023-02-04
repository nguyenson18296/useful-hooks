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

### useOnClickOutside
```javascript
export const useOnClickOutside = (ref: RefObject<HTMLDivElement>, handler: (e: Event) => void) => {
    useEffect(() => {
        const listener = (event: Event) => {
            if (!ref.current || ref.current.contains(event.target as Node)) {
                return;
            }

            handler(event);
        };
        document.addEventListener("mousedown", listener);
        return () => {
            document.removeEventListener("mousedown", listener);
        };
    }, [ref, handler]);
};
```
### usePrevious
```javascript
import { useEffect, useRef } from "react";

/**
 * Hook for tracking the previous value of the React component prop.
 * This is useful as a replacement for the componentWillReceiveProps lifecycle method.
 * See: https://reactjs.org/docs/hooks-faq.html#how-to-get-the-previous-props-or-state
 * @internal
 */
export const usePrevious = <T>(props: T): T => {
    const previousProps = useRef<T>(props);

    useEffect(() => {
        previousProps.current = props;
    });

    return previousProps.current;
};
```

### useDeepEffect
```javascript
import { useRef, useEffect } from "react";
import isEqual from "lodash/isEqual";

export const useDeepEffect = (effectFunc: () => any, deps: React.DependencyList) => {
    const isFirst = useRef(true);
    const prevDeps = useRef(deps);
    useEffect(() => {
        const isSame = prevDeps.current.every((obj, index) => isEqual(obj, deps[index]));
        if (isFirst.current || !isSame) {
            effectFunc();
        }
        isFirst.current = false;
        prevDeps.current = deps;
        // The static verification warning for deps makes no sense here
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, [effectFunc, ...deps]);
};
```

### ScrollGradient
```javascript
import { useState, useEffect } from "react";

export const useNumberState = () => useState<number>(0);

export function useContentHeight(element: HTMLElement | null) {
    const [contentHeight, setContentHeight] = useState(-1);

    useEffect(() => {
        let id = -1;

        function onFrame() {
            if (element && contentHeight !== element.scrollHeight) {
                setContentHeight(element.scrollHeight);
            }
            id = window.requestAnimationFrame(onFrame);
        }

        id = window.requestAnimationFrame(onFrame);
        return () => window.cancelAnimationFrame(id);
    }, [element, contentHeight, setContentHeight]);

    return contentHeight;
}

export function useScrollEvent(
    content: HTMLElement | null,
    size: number,
    onScroll?: (event: React.MouseEvent<HTMLDivElement>) => void,
): {
    top: number;
    bottom: number;
    onScrollHandler: (event: React.MouseEvent<HTMLDivElement>) => void;
} {
    const [top, setTop] = useNumberState();
    const [bottom, setBottom] = useNumberState();
    const contentHeight = useContentHeight(content);

    useEffect(() => {
        calculateOpacities(content, size, [top, setTop], [bottom, setBottom]);
    }, [bottom, setBottom, setTop, size, top, content, contentHeight]);

    const onScrollHandler = useCallback(
        (event: React.MouseEvent<HTMLDivElement>) => {
            calculateOpacities(content, size, [top, setTop], [bottom, setBottom]);
            onScroll?.(event);
        },
        [bottom, onScroll, setBottom, setTop, size, top, content],
    );

    return { top, bottom, onScrollHandler };
}

function calculateOpacities(
    content: HTMLElement | null,
    size: number,
    t: ReturnType<typeof useNumberState>,
    b: ReturnType<typeof useNumberState>,
) {
    const scrollTop = content ? content.scrollTop : 0;
    const topOpacity = calculateOpacity(scrollTop, size);

    const scrollBottom = content ? content.scrollHeight - content.offsetHeight - content.scrollTop : 0;
    const bottomOpacity = calculateOpacity(scrollBottom, size);

    const [top, setTop] = t;
    if (top !== topOpacity) {
        setTop(topOpacity);
    }

    const [bottom, setBottom] = b;
    if (bottom !== bottomOpacity) {
        setBottom(bottomOpacity);
    }
}

function calculateOpacity(current: number, size: number) {
    const opacity = Math.min(current / size, 1);

    if (opacity > 0) {
        return Math.max(opacity, 0.2);
    }
    return opacity;
}

```

### useFocus
 ```javascript

import React, { useEffect, useRef } from "react";

export function useFocus(component: HTMLElement | InputPure | null, focus: boolean): void {
    const timer = useRef<number>(-1);
    const element = getElement(component);

    useEffect(() => {
        clearInterval(timer);
        if (element && component && focus) {
            createInterval(
                timer,
                () => isVisible(element),
                () => component.focus(),
            );
        }

        return () => clearInterval(timer);
    }, [component, focus, element]);
}

function isVisible(element: HTMLElement | null) {
    if (element) {
        const style = window.getComputedStyle(element);
        const notHidden = style.visibility !== "hidden";
        const notNone = style.display !== "none";
        const hasSize = element.offsetHeight > 0;

        return notHidden && notNone && hasSize;
    }
    return false;
}

function getElement(component: HTMLElement | InputPure | null): HTMLElement | null {
    if (component instanceof InputPure) {
        return component.inputNodeRef || null;
    }
    return component;
}

function clearInterval(timer: React.MutableRefObject<number>) {
    window.clearInterval(timer.current);
}

function createInterval(timer: React.MutableRefObject<number>, check: () => boolean, done: () => void) {
    timer.current = window.setInterval(() => {
        if (check()) {
            done();
            clearInterval(timer);
        }
    }, 25);
}

 ```
