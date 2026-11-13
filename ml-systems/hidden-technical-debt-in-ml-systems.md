# Hidden Technical Debt in Machine Learning Systems

**Paper:** Sculley et al., NeurIPS 2015
**Read:** 12 Nov 2026 (Sections 1 to 3; Sections 4 to 5 next session)
**Topic:** ML systems in production

## Summary in 3 lines
- In a real ML system, the model code is a small box. Most of the work, and most of the risk, is in the data, pipelines, configuration and monitoring around it.
- ML systems pick up "hidden" technical debt because models depend on data and on each other in ways that normal code reviews and tests don't catch.
- That debt doesn't break things loudly. It makes them quietly worse, and it gets harder to pay off the longer it's ignored.

## Key ideas

### CACE: Changing Anything Changes Everything
A model mixes all its inputs together, so you can't change one part in isolation.
Example: adding a new feature (n+1) to an XGBoost model doesn't just add one more input. It changes which splits the trees choose and how important every other feature is, so the whole model's behavior shifts. Results might improve or get worse. Removing a feature or changing hyperparameters has the same effect.

**Explained without the tech:** It's like adding a new ingredient to a recipe. You don't just get the old dish plus one extra flavor. The salt, the cooking time and everything else might need to change too.

### Unstable data dependencies
An input whose *meaning* changes over time without the model knowing.
**My example (Newmark):** Property Keys are linked to buildings. Some keys are vetted and some aren't, and the vetting team keeps fixing them. So the same input means something different month to month.
- Bad data is a *data quality* problem. Data that keeps changing under the model is an *unstable dependency* problem.
- Even the fix can hurt. A model trained on partly wrong keys may have learned to work around the errors, so after vetting its predictions can get *worse* until it's retrained and tested.
- Models usually degrade quietly rather than break outright.

### Undeclared consumers
Teams that use a piece of data without its owners knowing. If the vetting team doesn't know the data science team relies on the keys, nobody warns them when the keys change.

### Stacked models / correction cascades
Feeding one model's output into another (e.g., an image classifier's output as a feature in a Cap Rate model) makes the second model depend on the first one's behavior. Improving the first model can break the second.

## What I'd do differently in a real project
1. **Version the data.** The model uses a frozen snapshot of the Property Keys. It switches to the newly vetted version only after retraining and testing on it.
2. **Define who tells whom.** Vetting team → Data Engineering → Data Science.
3. **Back the people with automation.** A version number on the keys table triggers an automatic alert when it changes, and monitoring on the inputs warns if the share of changed keys suddenly jumps.
4. **Write down who uses each data source,** so there are no undeclared consumers.
5. **Compare old vs new model** before shipping any feature change (because of CACE), not just the overall score.

## Questions I still have
- How do teams decide when it's worth paying down ML debt vs. shipping new features?
- What tools do companies use to version data and track who consumes it?
- (Add your own)

## Next
- Finish Sections 4 to 5: feedback loops, glue code, pipeline jungles
- Then: *Rules of Machine Learning* (Zinkevich)
