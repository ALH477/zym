# Design Specification for ZYM Package Manager (Revision 3)

## License
This design specification and the associated ZYM project are released under the BSD-3-Clause License:

Copyright (c) 2026 DeMoD LLC

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## 1. Overview
**Project Name:** ZYM (tentative; evoking "zymurgy" for transformative build processes, aligned with DeMoD LLC's innovative focus on fermentation-like iterative and evolving development cycles in software deployment).  
**Developer:** DeMoD LLC (led by @Im_Asher_LeRoy, emphasizing digital signal processing (DSP) for audio production, gaming environments, security applications, and open-source/hardware solutions).  
**Version:** 0.4 (Updated Draft, as of January 11, 2026, incorporating nixpkgs compatibility, autonomy requirements, integration of an SBCL SLIME Lisp DSL for enhanced expressiveness and interactive development, foundational support for flakes as a core feature for reproducibility and modularity, and the ability to create bootable images with packaged kernels for deployment flexibility).  
**Description:** ZYM is a clean-room reimplementation of a purely functional package manager, rooted in Eelco Dolstra's 2006 PhD thesis *"The Purely Functional Software Deployment Model"*. It serves as the foundational build and deployment system for the "Oligarchy" NixOS distribution, with optimizations tailored for DSP/audio workflows (e.g., low-latency plugin management), gaming (e.g., reproducible engine and mod deployments), and security (e.g., auditable supply-chain verification). Built entirely from scratch in Rust to ensure complete decoupling from existing Nix codebases, ZYM leverages modern language features for improved performance, concurrency safety, and maintainability. This revision integrates an SBCL (Steel Bank Common Lisp) and SLIME (Superior Lisp Interaction Mode for Emacs)-based Lisp DSL as an optional but powerful layer for derivation definitions, enhancing metaprogramming, interactivity, and domain-specific expressiveness while maintaining full nixpkgs compatibility and operational autonomy. The Lisp integration allows for dynamic, macro-driven configurations that align seamlessly with the thesis's functional purity, providing a bridge between Rust's static safety and Lisp's flexible, code-as-data paradigm. Additionally, flakes are now established as a foundational feature, serving as the primary mechanism for defining, locking, and composing reproducible configurations, with full semantic parity to Nix flakes for compatibility while introducing ZYM-specific extensions for autonomy. Finally, ZYM includes native support for creating bootable images (e.g., ISO, disk images, VM formats) that incorporate packaged kernels, enabling users to generate self-contained, reproducible system images tailored for Oligarchy's niches, such as embedded DSP devices or secure gaming environments.  
**License:** BSD-3-Clause (permissive to encourage commercial adoption, collaborations, and embeddings in proprietary tools, while avoiding the copyleft restrictions of LGPL-2.1+ used in Nix).  
**Repository:** (To be established; for example, on GitHub under the DeMoD LLC organization, with initial mirrors on self-hosted forges for added autonomy and resilience against platform dependencies).  
**Target Platforms:** Linux as the primary platform (core for the Oligarchy NixOS distribution), with built-in support for cross-compilation to macOS and Windows environments to facilitate development and testing in heterogeneous setups.

## 2. Goals and Motivations
- **Primary Objectives:**
  - Achieve functional parity with the core concepts of Nix (pure derivations, immutable store paths, reproducible builds) while introducing targeted improvements to address common pain points such as syntactic complexity, cryptic error messaging, and suboptimal performance in resource-intensive workflows like DSP processing.
  - **Nixpkgs Compatibility:** Enable seamless evaluation, building, and utilization of packages from the nixpkgs repository, allowing users to draw upon the extensive existing ecosystem without requiring forks, rewrites, or modifications to upstream expressions.
  - **Independent Autonomy:** Ensure ZYM functions as a fully standalone system with its own isolated store, evaluator, builder, and tooling suite, eliminating any runtime dependencies on Nix binaries, daemons, stores, or governance structures. This autonomy supports independent evolution, custom divergences for Oligarchy's needs, and resilience against upstream changes.
  - Provide ergonomic integrations with Rust's Cargo ecosystem, particularly for DSP/audio crates, to streamline development in audio production and gaming domains.
  - Incorporate an SBCL SLIME Lisp DSL to enhance expressiveness, enabling macro-based metaprogramming for concise domain-specific derivations (e.g., audio signal graphs) and interactive REPL-driven workflows for rapid prototyping and debugging.
  - Establish flakes as a foundational feature, making them the default and core paradigm for all configurations, derivations, and system definitions to ensure maximum reproducibility, composability, and lockfile-based pinning from the ground up.
  - Enable the creation of bootable images with packaged kernels, allowing users to generate reproducible, self-contained system images (e.g., for VMs, embedded devices, or installation media) that bundle custom or standard kernels with ZYM-managed packages, tailored for DSP, gaming, or security use cases.
- **Motivations (Based on Public Statements and Project Context):**
  - Deep frustration with the upstream Nix ecosystem, including community governance issues, licensing constraints, and accumulated technical debt, as expressed in statements like "I'm sick of it."
  - Need for a highly customized toolset in the Oligarchy NixOS distribution, such as integrated MIDI boot intros, specialized DSP plugin management, and security-focused reproducibility.
  - Clean-room methodology to mitigate legal and intellectual property risks, allowing for radical innovations without inheritance of legacy code.
  - Desire to fuse Rust's performance and safety with Lisp's flexibility, creating a hybrid system that excels in creative, iterative domains like DSP and gaming while grounding everything in Dolstra's thesis for theoretical soundness.
  - Integration of flakes as foundational to address Nix's historical fragmentation around reproducibility, providing a more robust starting point for Oligarchy's distro-specific needs.
  - Addition of image creation with packaged kernels to support deployment scenarios in Oligarchy, such as building custom live images for audio production setups or secure kernels for gaming hardware.
- **Non-Goals:**
  - Serving as a broad, drop-in replacement for Nix in general-purpose computing; instead, prioritize a curated ecosystem optimized for Oligarchy's niches.
  - Achieving immediate full parity with every edge case in nixpkgs; implement compatibility in phases, starting with core subsets and expanding based on user needs.
  - Mandating the Lisp DSL for all users; it remains optional to preserve simplicity for those preferring the Rust-embedded DSL.
  - Overcomplicating flake support with unnecessary extensions; focus on core flake principles with targeted ZYM enhancements.

## 3. Core Principles (Grounded in Dolstra's Thesis)
ZYM strictly adheres to the purely functional deployment model outlined in Dolstra's thesis:
- **Purity:** All builds are treated as side-effect-free functions that map fixed inputs (sources, dependencies, build scripts) to immutable outputs, ensuring no external state influences the results.
- **Immutability:** Artifacts are stored in a content-addressed filesystem, preventing modifications and enabling safe sharing across environments.
- **Reproducibility:** Given identical inputs, builds yield bit-for-bit identical outputs, which is crucial for DSP/audio applications where binary variations could introduce unintended artifacts or latency inconsistencies.
- **Lazy Evaluation:** Dependencies and builds are resolved and executed only when necessary, optimizing resource usage in large graphs.
- **Garbage Collection:** Automated removal of unused store paths, with configurable pinning mechanisms to protect critical derivations.

Extensions beyond the thesis to enhance usability and domain fit:
- **Concurrency Safety:** Utilize Rust's ownership and borrow checker to enable safe parallel evaluation and building without data races.
- **Modularity:** Design components as pluggable modules (e.g., custom resolvers or evaluators) to support extensions like the Lisp DSL.
- **Compatibility Principle:** Ensure behavioral equivalence with Nix for nixpkgs expressions, including matching evaluation semantics, hash computations, and store path formats where feasible.
- **Autonomy Principle:** All core operations must function without external dependencies; modular layers (e.g., Lisp bridge) are optional and isolatable.
- **Flakes Foundational Principle:** Flakes are the primary interface for all ZYM operations, enforcing lockfile-based input pinning, modular inputs/outputs, and reproducible evaluations as a built-in, non-optional feature to guarantee consistency across all derivations and system configurations.

## 4. Architecture
### 4.1 High-Level Components
- **Store:** A content-addressed filesystem (e.g., default path `/zym/store/<hash>-name`), employing BLAKE3 for hashing due to its superior speed and security over SHA-256. The store supports binary caches for efficient remote artifact fetching and distribution. **Autonomy:** The store format is fully independent, with no assumptions about Nix structures, but includes utilities for exporting/importing paths in Nix-compatible formats. **Compatibility:** Import tools allow pulling Nix store paths into ZYM, remapping them to local hashes if needed.
- **Evaluator:** A Rust-based interpreter for the primary embedded DSL, with semantic parity to Nix's expression language. **Compatibility:** Tested rigorously against nixpkgs expressions to ensure equivalent outcomes. **Autonomy:** The evaluator operates standalone, with pluggable backends for extensions. This includes integration with the SBCL SLIME Lisp DSL as an alternative or complementary layer (detailed in Section 4.5). Flakes are evaluated natively, with lockfiles parsed and enforced during all operations.
- **Builder:** A sandboxed execution engine using Rust's `std::process` combined with Linux namespaces, chroot, and seccomp filters for isolation. **Compatibility:** Capable of executing Nix-style build scripts without modification. **Autonomy:** Defaults to pure Rust-based sandboxing, with no reliance on external tools like Nix's daemon. Supports building kernel packages and integrating them into image creation workflows.
- **Resolver:** A hybrid dependency solver that combines Cargo's semantic versioning and feature resolution with fixed-output hashing for purity. **Compatibility:** Supports nixpkgs' sourcing primitives like `fetchurl` and `fetchgit`. **Autonomy:** Configurable to use ZYM-specific registries or mirrors. Flake inputs are resolved with lockfile enforcement.
- **CLI:** A user-friendly command-line interface with subcommands such as `zym build`, `zym install`, `zym shell`, `zym slime-repl` (for Lisp integration), `zym flake init`, and `zym image create`. Error messages are designed for clarity, including visual dependency graph outputs and suggestions for fixes.
- **Daemon:** An optional multi-user service for shared store management, with ZYM-native permission models emphasizing security. **Autonomy:** No inheritance from Nix's multi-user assumptions; fully self-contained.

### 4.2 Data Flow
1. The user defines a derivation using either the Rust-embedded DSL, a nixpkgs-imported expression, or the Lisp DSL (via SLIME or embedded interpreter), typically within a flake structure (e.g., `flake.zym` with inputs and outputs).
2. The resolver computes the full dependency graph, pinning all inputs with content hashes from the flake lockfile to ensure reproducibility.
3. The evaluator processes the definition: For nixpkgs compatibility, it operates in a strict "Nix-mode" mimicking Nix 2.x semantics; for Lisp derivations, it translates or directly evaluates via the SBCL bridge; all evaluations enforce flake semantics.
4. The builder executes the derivation in an isolated environment, producing immutable store paths that can be exported to Nix formats if required. For image creation, this step includes packaging kernels and assembling image artifacts.
5. Outputs are symlinked to user or system profiles for atomic activation, with rollback support. Flake outputs can include generated images.

### 4.3 Integration with Rust's Cargo
- **Cargo as Derivation Backend:** Cargo builds are wrapped as pure derivations, taking `Cargo.toml`, `Cargo.lock`, and vendored/hashed sources as inputs, and outputting store-installed binaries or libraries. Flakes can define Cargo-based outputs directly.
- **Hybrid Workflow Commands:** Commands like `zym cargo build` enforce purity by redirecting all mutable operations (e.g., `target/` directory) to temporary, isolated paths that feed into the store.
- **Ecosystem Synergy:** Automatic generation of derivations for crates.io packages, with a curated index for DSP-focused crates (e.g., `fundsp` for functional DSP graphs, `nih-plug` for audio plugins). This integration ensures low-latency optimizations are applied reproducibly, and flakes can lock Cargo dependencies.
- **Autonomy:** A standalone Cargo mode allows divergence from nixpkgs Rust overlays, enabling ZYM-specific features like custom build flags for gaming performance.

### 4.4 Compatibility Layer
- **Nixpkgs Integration:** 
  - Importer Tool: `zym import-nixpkgs <revision>` fetches a pinned revision of nixpkgs (via git clone or tarball), evaluates it within ZYM's environment, and makes packages available as if native. Flakes can wrap nixpkgs as inputs.
  - Evaluation Bridge: Nixpkgs expressions are run in a compatibility mode that emulates Nix's scoping, impurities (if opted-in), and function behaviors. Testing draws from projects like Tvix for inspiration.
  - Subset Focus: Initial compatibility targets core nixpkgs subsets (e.g., basic utilities, Rust toolchains), expanding to full coverage.
- **Store Interoperability:** 
  - Export/Import Utilities: Command-line tools to convert ZYM store paths to Nix-compatible formats (matching hash schemes, metadata serialization) and vice versa, facilitating hybrid setups.
  - Binary Cache Support: Fetch from caching.nixos.org or community mirrors, but with ZYM-hosted alternatives for autonomy.
- **Migration Path:** Provided scripts guide users from pure Nix to hybrid (ZYM evaluator with Nix store) to full ZYM autonomy, minimizing disruption. Flakes simplify migration by allowing mixed inputs.
- **Divergence Handling:** Flags like `--nix-compat` enforce strict Nix semantics; without it, ZYM extensions (e.g., enhanced DSP primitives) can be applied.

### 4.5 Integration of SBCL SLIME Lisp DSL
A Lisp-based Domain-Specific Language (DSL) implemented via SBCL (Steel Bank Common Lisp) and integrated with SLIME (Superior Lisp Interaction Mode for Emacs) is incorporated as an optional layer to enhance ZYM's derivation language, expressiveness, and developer workflow. While ZYM's core remains Rust-based for performance, safety, and systems-level operations, the Lisp DSL provides a dynamic, metaprogrammable frontend that aligns with the purely functional model from Dolstra's thesis—leveraging Lisp's functional paradigms (immutability, higher-order functions, closures) for natural representation of derivations. Flakes can be defined in Lisp syntax, with macros generating flake-compatible structures.

#### 4.5.1 Enhanced Expressiveness and Metaprogramming for Derivations
- **Macro System Power:** Lisp's homoiconicity (where code is treated as data) enables powerful macros for creating concise, domain-specific syntax. In ZYM, this allows users to define complex derivations with minimal boilerplate. For example, a macro could define DSP pipelines as `(define-audio-graph (input :mic) (filter :reverb :params (list :decay 0.5)) (output :jack))`, which expands at evaluation time into a reproducible build graph with hash-pinned dependencies on audio libraries. This reduces complexity compared to Nix's flat expression language or ZYM's base Rust DSL, making it ideal for modeling interconnected systems like audio effects chains, gaming mod compositions (e.g., procedural asset generation), or security policy graphs (e.g., layered access controls). Flakes can use Lisp macros to define inputs/outputs dynamically.
- **Benefit to Oligarchy's Niches:** In DSP/audio domains, the Lisp DSL integrates with libraries like Incudine for real-time sound synthesis, enabling derivations that include "live" prototyping elements—e.g., embedding signal processing code directly in the derivation for preview during evaluation. For gaming, macros can generate derivations for shader compilations or engine configurations. In security, macros facilitate audit rule generation, such as creating reproducible firewall or encryption setups with embedded verification logic. Image creation can be macro-ized, e.g., `(define-image :kernel "linux-6.5" :packages (list :dsp-tools) :format :iso)`.
- **Comparison to Current Design:** The base Rust-embedded DSL provides simplicity and static checks, but the Lisp layer adds user-extensible syntax without requiring core modifications. This promotes autonomy by allowing community-driven extensions and maintains nixpkgs compatibility through transpilation (Lisp macros expand to Nix-equivalent expressions).

#### 4.5.2 Interactive Development and REPL-Driven Workflow
- **SLIME Integration:** SLIME offers an Emacs-based REPL with features like live code evaluation, debugging, inspection, and completion. ZYM users can launch interactive sessions via `zym slime-repl`, loading a Lisp environment to define, test, and refine derivations dynamically. For instance, a developer could load a partial flake, evaluate it to visualize the dependency graph, tweak parameters (e.g., DSP filter coefficients), and iterate without triggering full rebuilds. This is particularly valuable for DSP debugging, where real-time audio previews (via integrated backends) can be heard during sessions.
- **Workflow Boost:** The REPL supports prototyping Oligarchy configurations interactively—e.g., evaluating a MIDI boot intro derivation within a flake, inspecting store paths, generating images with packaged kernels, and performing on-the-fly garbage collection. This interactivity surpasses Rust's static compile-run cycle, accelerating creative workflows in audio production, game modding, or security policy tuning.
- **Autonomy Edge:** The Lisp layer decouples experimentation from the Rust core, allowing independent updates to the DSL without affecting ZYM's foundational components, including flake handling.

#### 4.5.3 Reproducibility and Functional Purity Alignment
- **Thesis Synergy:** Dolstra's model focuses on pure functions; Lisp's functional style, enhanced by SBCL's optimizations for immutability and efficient closures, ensures derivations remain side-effect-free. Lisp closures can natively represent lazy derivations, with SBCL compiling to machine code for performance. Flakes lockfiles are serialized in Lisp for reproducible evaluations.
- **Nixpkgs Compatibility:** A bidirectional translator handles conversions: Lisp to Nix expressions (for importing nixpkgs into Lisp mode) and Nix to Lisp (for exporting ZYM configs). This preserves reproducibility—e.g., evaluate nixpkgs in Lisp, then build via ZYM's Rust backend. For autonomy, a pure Lisp evaluation mode operates without Nix interop.
- **Performance in DSP/Gaming:** SBCL's high optimization levels make Lisp-evaluated derivations efficient for real-time previews, with Rust managing heavier tasks like store I/O or image assembly.

#### 4.5.4 Ecosystem and Tooling Integration
- **Cargo Synergy:** Lisp macros can embed Rust crate definitions, e.g., `(cargo-crate "fundsp" :version "0.9" :features (:dsp :realtime))`, creating hybrid workflows: SLIME for orchestration, Cargo for builds. Flakes can include Cargo-locked inputs.
- **Community and Extensibility:** Leverage Lisp's ecosystem (e.g., Quicklisp for package loading) to incorporate DSP tools like CLM (Common Lisp Music), positioning Oligarchy as attractive to Lisp-savvy developers. SLIME's Emacs integration appeals to Emacs/Nix users, expanding adoption without core compromises.
- **Security Benefits:** Lisp's dynamic introspection allows runtime scanning of macro-expanded code for vulnerabilities, bolstering ZYM's audit tools, including flake input verification.

#### 4.5.5 Potential Integration Approaches
- **Embedded Interpreter:** Use Rust crates like `lisp-rs` or a custom FFI to embed a minimal Lisp runtime, with SBCL as an optional external process for full features. Lisp derivations are parsed and evaluated in Rust, outputting to the store, with flake structures handled natively.
- **Bridge Mode:** The ZYM CLI spawns SBCL with SLIME, communicating via pipes or sockets (e.g., JSON for serialization). This keeps the core lightweight while supporting advanced interactivity, including flake editing.
- **Phased Rollout:** Introduce in Phase 2 (Q2 2026), starting with basic macros for DSP; test against nixpkgs subsets. Make it pluggable—users can configure via flags to use pure Rust DSL instead.

#### 4.5.6 Trade-Offs and Considerations
- **Challenges:** Lisp's dynamic nature may add evaluation overhead; mitigate with SBCL's ahead-of-time (AOT) compilation and caching. Integration complexity (Rust-Lisp FFI) could extend timelines, so scope to an MVP with core macros. Community adoption might be niche, but it differentiates ZYM from Nix/Lix/Tvix.
- **Why Beneficial Overall:** This integration transforms ZYM from a functional reimplementation into an innovative, empowering platform. Lisp addresses Nix's syntactic rigidity, making it suited for creative, iterative domains. It upholds autonomy (no Nix code dependency) and compatibility (via translation), turning ecosystem frustrations into a fresh toolset.

## 5. Features
### 5.1 General
- **Flakes as Foundational Feature:** Flakes are the core paradigm for all ZYM configurations, providing modular inputs (e.g., pinned git revisions, URLs), outputs (e.g., packages, systems, images), and automatic lockfile generation (`zym.lock`) for reproducibility. Unlike Nix, where flakes are experimental, ZYM treats them as mandatory for top-level definitions, with commands like `zym flake init` creating templates. **Compatibility:** Full semantic parity with Nix flakes, allowing direct use of nixpkgs flakes as inputs. **Autonomy:** ZYM extensions include domain-specific flake schemas (e.g., for DSP graphs) and optional impurities for prototyping. Flake evaluation is lazy and pure, integrating with the evaluator for seamless operation.
- **Cross-Compilation:** Built-in mechanisms for targeting embedded DSP hardware (e.g., ARM-based audio devices) or gaming platforms, with flakes defining cross-build outputs.
- **Profiles and Rollbacks:** Atomic switches between user/system profiles, with generation tracking for easy reversion. Flakes can define profile outputs.
- **Image Creation with Packaged Kernels:** Native support for generating bootable images (e.g., ISO, QCOW2 for VMs, raw disk images) that bundle ZYM-packaged kernels with selected packages. Commands like `zym image create --kernel <derivation> --format iso --packages <list>` assemble images from flake outputs, ensuring reproducibility via locked inputs. Kernels are treated as derivations (e.g., custom-patched Linux for low-latency DSP), with tools for initrd generation, bootloader integration (e.g., GRUB or systemd-boot), and file system population. **Compatibility:** Mirrors NixOS image builders for nixpkgs interop. **Autonomy:** ZYM-specific formats for embedded devices, with security features like signed images.

### 5.2 Domain-Specific (DSP/Audio/Gaming/Security)
- **Audio Optimizations:** Native support for derivations involving JACK, ALSA, or PipeWire; reproducible plugin builds (VST/CLAP/LV2) with automated latency profiling and optimizations. Flakes can define audio stacks as outputs, and images can package real-time kernels for live audio.
- **Gaming Focus:** Integration with tools like Proton/Wine for reproducible game environments; derivation-based mod management to ensure consistent deployments. Flakes lock game dependencies, and images create bootable gaming ISOs with optimized kernels (e.g., for reduced input lag).
- **Security Enhancements:** Mandatory input hash pinning (enforced via flakes); built-in audit tools for supply-chain verification; sandboxed builds with advanced seccomp filters for isolation. Images support secure boot with packaged signed kernels.
- **Performance:** Leverage Rust's async I/O (via Tokio) for faster dependency fetches and parallel builds; Lisp DSL adds interactive performance tuning. Image creation parallelizes kernel packaging and assembly.

### 5.3 Differences from Nix/Lix/Tvix
- **Vs. Nix/Lix:** ZYM's clean-room Rust implementation provides inherent memory safety and concurrency advantages over C++; BSD-3-Clause license offers more flexibility; narrower scope focuses on Oligarchy niches rather than broad ecosystems. Flakes are foundational (not optional), and image creation is integrated with kernel packaging.
- **Vs. Tvix:** Shares Rust/clean-room ethos but diverges in focus—ZYM prioritizes Cargo and Lisp integrations for DSP/gaming, while borrowing Tvix's nixpkgs testing strategies. ZYM adds explicit autonomy tools like store exporters, pluggable DSLs, foundational flakes, and image/kernel features.
- **UX Improvements:** Simplified syntax in base DSL; Lisp layer for advanced users; better domain-specific primitives (e.g., audio graph builders) that can be toggled for nixpkgs compatibility. Flake-first design reduces configuration fragmentation.

## 6. Implementation Plan
- **Phase 1 (Q1 2026):** Develop core store, basic evaluator, and builder; implement foundational flakes (lockfile parsing, input resolution) and initial nixpkgs expression evaluation (focus on simple examples like "hello world" and Rust crates) to validate compatibility.
- **Phase 2 (Q2 2026):** Add CLI, Cargo integration, nixpkgs importer, and autonomy tools (e.g., standalone operation modes); introduce SBCL SLIME Lisp DSL as an optional layer, with basic macros and REPL support tested on DSP subsets; integrate image creation with basic kernel packaging.
- **Phase 3 (Q3 2026):** Expand to full compatibility testing (build and verify 100+ nixpkgs packages); curate initial DSP/gaming/security package sets; enhance flakes with ZYM extensions; add advanced image formats and kernel customization; conduct security audits and performance benchmarks.
- **Dependencies:** Rust 1.75+; key crates including `blake3` for hashing, `tokio` for async operations, `serde` for serialization, `clap` for CLI parsing; optional nix-compat libraries for bridging during migration; SBCL for Lisp DSL, with FFI crates like `rust-sbcl-ffi` (hypothetical or custom); additional crates for image handling (e.g., `squashfs-tools-rs` for file systems, `bootloader-rs` for kernels).
- **Testing:** Unit tests for purity and isolation; integration tests against Dolstra's thesis examples and nixpkgs subsets; fuzzing for resolver and evaluator robustness; specific Lisp tests for macro expansion and REPL stability; flake-specific tests for lockfile integrity; image tests for bootability (e.g., QEMU emulation). **Compatibility Benchmark:** Daily CI runs evaluating nixpkgs master to detect regressions.
