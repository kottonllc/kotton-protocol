# RFCs

An RFC is a change to the protocol, or a decision that binds more than one of Kotton's
repositories: what a process is, what a request means, what crosses a boundary, which
repository owns what. A decision that binds one repository's code alone lives in that
repository, beside the code.

## How one lands

1. **A pull request adds a file here**, numbered next in order, named for what it decides:
   `NNNN-what-it-decides.md`. It has three sections: *Context* (what is true now, and why it
   is not enough), *Decision* (what will be true, stated so a stranger could implement it),
   *Consequences* (what it costs, what it makes possible, what it leaves open).
2. **It names what it reaches**: the repositories whose code changes, and the version of the
   protocol the change ships in, when it changes the protocol.
3. **An assessor and a reviewer read it** before it merges, as for any pull request: does it
   decide what it says, does it keep every principle, does it leave a seam toward the north
   star where it concedes something.
4. **Once merged, it is not edited.** A later RFC supersedes it and says so. Before it merges,
   it may be rewritten freely.
5. **Each repository it reaches** gets an issue naming the RFC, and the change lands there
   through that repository's own loop. The protocol's change lands here first, tagged; the
   dependents take the tag.

A page under `design/` describes the whole as it is; an RFC describes a change to it. When
an RFC merges, the page it makes untrue is corrected in the same pull request.
