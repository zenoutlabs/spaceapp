# SpaceApp memory graph — measured evidence

Candidate [`c765ee7c490a177f8eda93a7e1546b5e74595192`](https://github.com/zenoutlabs/spaceapp/commit/c765ee7c490a177f8eda93a7e1546b5e74595192), base [`adb96ac77531649bd335078aacaa0318ce1d20a2`](https://github.com/oll4com/spaceapp/commit/adb96ac77531649bd335078aacaa0318ce1d20a2). The candidate changes exactly the 35 source paths in [SOURCE_MANIFEST.json](SOURCE_MANIFEST.json); each published blob hash was independently verified. This bundle records local observations, not sponsor acceptance or payment. **Linux validation was pending when this bundle was prepared; no Linux pass is claimed.**

## Final production browser observations

Named hardware: MacBook Pro, MacBookPro18,2, Apple M1 Max integrated GPU, 64 GB RAM. Chrome 155 / ANGLE Metal; hardware concurrency 10. All normal runs use 1173.7578125 × 688 CSS px, DPR 2, titles enabled, actual WebGL `antialias: false` with the candidate’s analytic antialiasing, 30 warmup frames and 180 measured draws. The receipts retain all timing samples, actual context attributes, browser/GPU metadata, focus/visibility, and long-task entries.

| Synthetic graph | RAF interval p95 | Max sampled pick | Max observed load task | Receipt |
|---|---:|---:|---:|---|
| 2,000 nodes / 6,000 edges | 9.0 ms | 0.3 ms | **192 ms** | [2k](final-coordinate-2000-production-20261019-2010.json) |
| 10,000 / 30,000 | 9.3 ms | 1.2 ms | 0 ms / no ≥50 ms entries | [10k](final-coordinate-10000-production-20261019-2010.json) |
| 50,000 / 150,000 | 25.4 ms | 0.9 ms | 74 ms | [50k](final-coordinate-50000-production-20261019-2010.json) |
| 50,000 / 150,000, same-page interaction repeat | 25.1 ms | 0.3 ms | Same original load: 74 ms | [50k repeat](final-coordinate-50000-repeat-20261019-2010.json) |

[SpaceApp issue #62](https://github.com/oll4com/spaceapp/issues/62) specifies p95 ≤16.7 ms at 2k and ≤33 ms at 50k, and picking ≤4 ms / no load-layout task >100 ms specifically at 50k. These final observations meet those size-specific numeric bounds on this machine. They do not establish a universal upper bound for every future run or data distribution.

**Preserved limitations and pauses:**

- The two 50k reports share **one** load window: 11256.800000190735–332698.60000038147 ms, about **321.44 seconds** through full convergence. The 74 ms value is the longest observed main-thread task, not total loading time. The repeat is not a second independent load.
- At 2k, the full 192 ms task began at 19188.900000095367 ms and overlapped the graph-load marker at 19336.700000286102 ms. Its full duration remains in the report; it has not been clipped to the overlapping portion. Do not claim that all sizes load without a >100 ms task.
- The first final 50k interaction run contains a **59 ms Long Task**, maximum RAF interval 42.4 ms, and maximum CPU/submission duration 44 ms. The repeat has no ≥50 ms interaction task entries. A p95 pass is not a claim that every frame is below the threshold.
- RAF intervals measure foreground callback pacing, not proof each frame reached the display. CPU/submission timing is not GPU time. `NORMAL_INTERACTION` does not insert `gl.finish()`; the immutable harness also exposes a separately labeled completion-fenced diagnostic.
- Browser Long Task absence means no entries at its approximately 50 ms reporting threshold; it does not mean tasks took zero time.
- Earlier source/control runs included 11.2 ms max picking / 108 ms load, a diagnostic 398 ms load task, and other failures. The final targeted fixes followed those observations. This bundle is the latest-source matrix; it does not erase earlier audit receipts or assert a baseline-wide pass.
- Browser data is deterministic synthetic demo data with no backend/provider call or private user memory. It is not an end-to-end production backend/network measurement.

The [streaming input receipt](final-coordinate-streaming-input-20261019-2010.json) shows exact 50k/150k drawn while step 6 / layout complete false, with rotation changed during convergence. It demonstrates use before convergence, not a measured input-latency bound. The [pending-camera receipt](final-coordinate-50000-preredraw-20261019-2010.json) retains all 20 picks (maximum 0.9 ms): each queries the last displayed frame while the next camera change awaits redraw; index rebuilding occurs in draw, not in the picking timing.

The [context-loss/recovery receipt](final-coordinate-50000-recovery-20261019-2010.json) records the hidden/untabbable WebGL canvas and explicit bounded fallback status after loss, followed by a visible/tabbable restored GPU canvas with exact 50k/150k, busy false, and layout complete / step 180. This receipt does not itself count fallback draw calls.

## Server-only API measurement

[api-overview-50k-measurement.json](api-overview-50k-measurement.json) was collected at 19:16:39 UTC, before subsequent web-only performance fixes; the API implementation was unchanged by those fixes. It uses the real authenticated overview route and an **in-memory synthetic** 50k-node/150k-edge fixture: one cold request plus five warm sequential Fastify injections. The uncompressed response is **23,778,336 UTF-8 bytes**; cold duration **514.982459 ms**, warm median **417.135708 ms**, warm maximum **433.216958 ms**. All six samples and host load values remain in the receipt.

The boundary includes filtering, selection, redaction, contract validation, and serialization. It excludes snapshot construction, process startup, network transfer and browser JSON processing. `passed: true` means the script’s status/count/truncation/redaction checks passed, not that the entire repository suite or a server latency threshold passed. This earlier server receipt and the final demo browser matrix are different measurements and must not be combined into a claimed end-to-end production timing.

## Reproduce with an existing developer checkout

Use Node >=22.12 and <23, the existing installed repository dependencies and built workspace prerequisites. No package installation, provider credentials, Docker, backend account, or spending is needed for the synthetic demo steps below. Run from the repository root at the candidate commit.

```sh
npm run build -w @space/web
npm exec -w @space/web -- vite preview --host 127.0.0.1 --port 4911
```

Open `http://127.0.0.1:4911/demo-workspace?memoryGraphNodes=50000&memoryGraphBenchmark=1` and open the Memory workspace. The query selects the synthetic 3D size. Keep the tab foreground and focused, use the receipt’s viewport/DPR where comparing results, and allow layout to finish until the benchmark buttons enable. The 50k convergence above took several minutes; the map can be rotated and explored while it runs. Click **Run graph benchmark**, then copy its entire JSON. The harness resets the camera/focus to the stated trajectory and restores the prior view afterward. Avoid other interaction during measurement.

Use separate fresh navigations with `memoryGraphNodes=2000` and `memoryGraphNodes=10000` for their load windows. Repeating the benchmark on the same page repeats interaction only, not loading. **Measure picks before redraw** records the separate 20-sample diagnostic. **Test context loss** followed by **Restore WebGL context** checks fallback/recovery; restoration is subject to browser permission. **Measure WebGL completion** is a separate diagnostic and must not be mislabeled as the ordinary-interaction path.

For functionality during development, `npm run dev:web` exposes the same demo URL. Development/HMR timing is a different boundary and must not replace production-preview results.

With the existing built API workspace prerequisites, reproduce the server-only fixture measurement separately:

```sh
node --import tsx apps/api/scripts/measure-memory-graph-overview.ts api-overview-50k-measurement.json
```

Immutable implementation links: [benchmark and lifecycle](https://github.com/zenoutlabs/spaceapp/blob/c765ee7c490a177f8eda93a7e1546b5e74595192/apps/web/src/features/memory-workspace/MemoryGraph3D.tsx), [WebGL renderer](https://github.com/zenoutlabs/spaceapp/blob/c765ee7c490a177f8eda93a7e1546b5e74595192/apps/web/src/features/memory-workspace/memory-webgl-renderer.ts), [spatial picking](https://github.com/zenoutlabs/spaceapp/blob/c765ee7c490a177f8eda93a7e1546b5e74595192/apps/web/src/features/memory-workspace/memory-pick-index.ts), [synthetic generator](https://github.com/zenoutlabs/spaceapp/blob/c765ee7c490a177f8eda93a7e1546b5e74595192/apps/web/src/demo/demo-memory-graph.ts), [API measurement script](https://github.com/zenoutlabs/spaceapp/blob/c765ee7c490a177f8eda93a7e1546b5e74595192/apps/api/scripts/measure-memory-graph-overview.ts).

## Integrity and privacy

Implementation and evidence preparation used AI coding assistance. This evidence bundle does not assert account-holder personal review or sponsor approval.

`SHA256SUMS` covers all other files in this bundle. The nine raw receipt/hardware JSON files were inspected for private absolute paths, email addresses, user/account identifiers and credential-like strings; none were found, so those nine files are byte-for-byte copies with **no redactions**. The public source manifest contains only public repository identity, commit/base, relative source paths, sizes and hashes; the original local absolute repository-directory value was replaced with the public repository URL. No bulk logs, session tokens, real user payloads or private source paths are included.
