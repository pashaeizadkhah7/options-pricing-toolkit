**Live demo:** https://pashaeizadkhah7.github.io/options-pricing-toolkit/
# Options Toolkit — Premium Pricer & Implied Volatility Solver

A single-file, zero-dependency options analytics tool that runs entirely in your browser.
Open `index.html` and you have a full American/European options pricer and IV solver —
no installs, no accounts, no data leaves your machine.

## Features

### Premium Pricer
- **5 strike slots side by side**, each with its **own implied volatility** input — so you
  can mirror the real volatility smile from your broker's option chain
- Any risk-free rate and dividend yield (typed inputs, no artificial caps)
- Three prices per strike: **American** (Leisen-Reimer binomial + Richardson extrapolation,
  the primary price), **European** (Black-Scholes-Merton), and **Bjerksund-Stensland 2002**
  closed-form as an independent cross-check
- Full Greeks per strike — Delta, Gamma, Theta/day, Vega/1%, Rho/1% — in either
  American (tree-consistent numerical) or European (analytical) mode
- Early-exercise premium, intrinsic/time value split, break-even at expiry
- Interactive profit/loss-at-expiry chart with hover readout for all 5 strikes
- Per-strike no-arbitrage checks and a cross-strike monotonicity check that flags
  IV combinations implying static arbitrage
- Optional discrete cash dividend (escrowed method)

### IV Solver
- Inverts the **American model price** (not just Black-Scholes) — the European B-S IV
  is shown alongside for comparison
- European analytic seed → damped Newton with model-consistent vega → Brent failsafe;
  typical convergence in a handful of iterations to a price residual below 10⁻⁸
- No-arbitrage pre-checks with plain-English explanations (e.g. premium below intrinsic)
- Convergence-quality score, model Greeks at the solved IV, and a ±20% volatility
  sensitivity band
- Honest handling of degenerate cases: options pinned at intrinsic value have no
  identifiable IV, and the tool says so instead of printing a misleading number

### Validation Suite
A built-in, one-click test suite (third tab) that checks the engine against:
- Textbook Black-Scholes reference values (10.450584 / 5.573526 for the classic
  S=K=100, T=1y, r=5%, σ=20% case)
- European put-call parity to machine precision
- American ≥ European no-arbitrage ordering; American call = European call when q=0
- Binomial-tree convergence (Richardson extrapolation vs a 2001-step tree)
- Closed-form BS2002 vs the tree
- 12 full round-trip IV recoveries (price at a known σ → invert → recover σ)

## Models & conventions

| Piece | Method |
|---|---|
| American pricing | Leisen-Reimer binomial tree (Peizer-Pratt method-2 inversion, odd steps) with Richardson extrapolation across two tree sizes |
| European pricing | Black-Scholes-Merton with continuous dividend yield |
| Closed-form cross-check | Bjerksund-Stensland (2002) two-boundary approximation; puts via the exact put-call transformation |
| Normal CDF | Cody (1969) rational minimax erfc (~15 significant digits) |
| Bivariate normal CDF | Drezner-Genz Gauss-Legendre quadrature (verified against brute-force integration to ~10⁻⁶) |
| Discrete dividends | Escrowed method: spot reduced by the PV of the dividend when the ex-date precedes expiry |
| IV root-finding | European analytic seed → damped Newton (numerical vega) → Brent bracketing failsafe |

Conventions: **T = calendar days / 365** (ACT/365), rates and yields are annualized and
continuously compounded, theta is per calendar day, vega and rho are per 1 percentage
point, prices are per share (×100 for a standard US equity contract).

## Why the numbers may differ from other calculators

Several widely-circulated JavaScript implementations of these formulas (including the
predecessors of this tool) contain three subtle transcription bugs that this project
fixes and guards with tests:

1. **Cody erfc coefficient order** — the netlib CALERF recurrence puts the leading
   polynomial coefficient *last* in the table; evaluating the table in listed order
   silently costs ~3 digits of accuracy in the normal CDF (≈ $0.08 on a $10.45 option).
2. **Drezner-Genz quadrature scale** — the Gauss-Legendre sum spans both half-intervals,
   so the correction term must be divided by 4π, not 2π.
3. **Genz tail-probability convention** — Genz's `BVND` returns the *upper* tail
   P(X>a, Y>b), not the CDF; using it directly breaks every bivariate-normal formula
   (this is why Bjerksund-Stensland "never worked" in many hobby implementations).

The validation tab exists so you can re-verify all of this in one click, in your own
browser, any time.

## Usage

1. Download `index.html` (or clone the repo)
2. Open it in any modern browser — that's it

To publish your own copy: enable **GitHub Pages** in the repo settings
(Settings → Pages → Deploy from branch → `main`, root folder) and the tool will be
live at `https://<your-username>.github.io/<repo-name>/`.

## Accuracy notes & honest limitations

- The binomial tree agrees with high-resolution references to well under a cent on
  standard contracts; the pricing bottleneck in practice is *your inputs* (IV, rate,
  dividends), not the numerics.
- Constant-volatility models are approximations. Real markets have volatility smiles,
  term structure, and discrete dividends — enter per-strike IVs from a live option
  chain for the most realistic outputs.
- Deep in-the-money American options can be pinned at intrinsic value, where IV is
  mathematically not identifiable. The solver flags this rather than inventing a number.
- This is an educational and research tool. **It is not financial advice, and no model
  output is a guarantee of market behavior.**

## License

MIT — see [LICENSE](LICENSE).
