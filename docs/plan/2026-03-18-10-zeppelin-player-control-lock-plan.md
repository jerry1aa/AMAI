## Respect player-control locks for Zeppelin transport logic

- Guard `ZEPPELIN_MOVE` against both the transport and the passenger being player-selected/locked.
- Guard `ZEPPELIN_FOLLOW` against the follow Zeppelin and the major hero being player-selected/locked.
- Filter follow-Zeppelin candidate selection by AI-controllable status.
- Guard the direct expansion transport helper `BuildMovePeonZeppelin` so selecting the peon or Zeppelin prevents AI transport orders.
- Update `CHANGELOG.md` and rebuild with `MakeTFT.bat`.
