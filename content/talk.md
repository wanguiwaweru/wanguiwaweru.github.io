## Abstract
What does it look like to build an audio DSP pipeline in Rust?

In this talk, we will go from raw audio samples to frequency-domain features using a composable, high-performance pipeline. Along the way, we implement core primitives like windowing and FFTs, and explore how to structure processing stages without unnecessary allocations.

We’ll look at zero-copy data flow, trait-based design, and the tradeoffs between batch and real-time systems. The talk also includes performance comparisons with Python-based approaches, and practical lessons from optimizing real pipelines.

If you’re interested in systems programming, signal processing, or just making things fast without giving up safety, this talk is for you.

## Description
This talk explores how to build audio DSP pipelines in Rust, based on experience implementing high-throughput audio feature extraction systems.

We’ll walk through core stages like windowing, FFT, and feature extraction, and learn how to compose them into reusable pipelines using traits and zero-copy data flow. The focus is on practical design: avoiding allocations in hot paths, structuring pipelines for batch and real-time workloads, and keeping systems fast and maintainable.

Along the way, I’ll share benchmarks against Python-based approaches, as well as mistakes and tradeoffs encountered when optimizing performance.

Attendees will leave with concrete patterns for building efficient DSP systems in Rust and a clearer sense of when Rust is a good fit for audio processing.

## Bio
I am a software engineer focused on high-performance systems and machine learning infrastructure. My work explores building scalable pipelines for audio processing, embedding systems, and similarity search. I have experience developing developer tools and I am particularly interested in using Rust to build fast, reliable systems for data-intensive applications.