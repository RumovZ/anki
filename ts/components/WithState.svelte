<!--
Copyright: Ankitects Pty Ltd and contributors
License: GNU AGPL, version 3 or later; http://www.gnu.org/licenses/agpl.html
-->
<script lang="typescript" context="module">
    import { writable } from "svelte/store";

    type UpdaterMap = Map<string, (event: Event) => boolean>;
    type StateMap = Map<string, boolean>;

    const updaterMap = new Map() as UpdaterMap;
    const stateMap = new Map() as StateMap;
    const stateStore = writable(stateMap);

    function updateAllStateWithCallback(callback: (key: string) => boolean): void {
        stateStore.update((map: StateMap): StateMap => {
            const newMap = new Map() as StateMap;

            for (const key of map.keys()) {
                newMap.set(key, callback(key));
            }

            return newMap;
        });
    }

    export function updateAllState(event: Event): void {
        updateAllStateWithCallback((key: string): boolean =>
            updaterMap.get(key)!(event)
        );
    }

    export function resetAllState(state: boolean): void {
        updateAllStateWithCallback((): boolean => state);
    }

    function updateStateByKey(key: string, event: Event): void {
        stateStore.update((map: StateMap): StateMap => {
            map.set(key, updaterMap.get(key)!(event));
            return map;
        });
    }
</script>

<script lang="typescript">
    export let key: string;
    export let update: (event: Event) => boolean;

    let state: boolean = false;

    updaterMap.set(key, update);

    stateStore.subscribe((map: StateMap): (() => void) => {
        state = Boolean(map.get(key));
        return () => map.delete(key);
    });

    stateMap.set(key, state);

    function updateState(event: Event): void {
        updateStateByKey(key, event);
    }
</script>

<slot {state} {updateState} />
