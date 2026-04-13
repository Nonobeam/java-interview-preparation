## Q4: NESTED propagation — does it only rollback the nested, not the parent?

When using `Propagation.NESTED`, if the nested method fails, does it only roll back the nested method's work? What happens if the parent fails instead?
