# geno-bench

Benchmark creation system that mines coding agent session logs for failure patterns and turns them into evaluation tasks.

## Overview

Coding agents generate detailed session logs (JSONL) containing every tool call, error, retry, and back-and-forth. geno-bench provides scripts to scan those logs, surface sessions with evidence of agent failure, extract per-session failure patterns, and organize findings into benchmark task designs.

## Navigation

- [Getting Started](getting-started.md) -- install and run your first session analysis
