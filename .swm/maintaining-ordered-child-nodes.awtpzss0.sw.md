---
title: Maintaining Ordered Child Nodes
---
This document describes how new child nodes are inserted into a parent node while maintaining the correct order. By comparing order values, the system ensures each new node is placed in the appropriate position, supporting consistent organization and display of data.

# Where is this flow used?

This flow is used multiple times in the codebase as represented in the following diagram:

(Note - these are only some of the entry points of this flow)

```mermaid
graph TD;
      1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> 56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(ui/…/bigtrace/index.ts::addNetworkSummary)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> 75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(ui/…/bigtrace/index.ts::addDeviceState)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> bda3161d36863754d81d970e5e6d51afe41fcb86f9f05bdd65a8ba1cbbfd91d7(ui/…/bigtrace/index.ts::addHighCpu)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> cbd75d0cd27c846937eb51087608a25cfb59af15b4cb872a4018155bee70c94f(ui/…/bigtrace/index.ts::addAtomCounters)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> 8824ae128b64a8cfdcf585e73b0182936397a50ca6de49ba8b36a4f5823eb820(ui/…/bigtrace/index.ts::addAtomSlices)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> 931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(ui/…/bigtrace/index.ts::addModemMintData)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> b554d4e13f81a5711eabf94baf7204fb3d0384e10884576efefc6dcce8ec8c08(ui/…/bigtrace/index.ts::addKernelWakelocks)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> 8c3f396c3b197b8a51586f0028985419207cf18e362e7a10476b559f9390f14a(ui/…/bigtrace/index.ts::addKernelWakelocksStatsd)

1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(ui/…/bigtrace/index.ts::onTraceLoad) --> 35ece870e0bd27cf5ef389cba4be5d5cff4c191cbd8483c175f6edf423020991(ui/…/bigtrace/index.ts::addWakeups)

56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(ui/…/bigtrace/index.ts::addNetworkSummary) --> e2e80e4b100e8dcc4fb98fd036ee752172fa1be983fea9645909304414f45dd6(ui/…/bigtrace/index.ts::addBatteryStatsState)

56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(ui/…/bigtrace/index.ts::addNetworkSummary) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(ui/…/bigtrace/index.ts::addNetworkSummary) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack)

56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(ui/…/bigtrace/index.ts::addNetworkSummary) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(ui/…/bigtrace/index.ts::addCounterTrack)

e2e80e4b100e8dcc4fb98fd036ee752172fa1be983fea9645909304414f45dd6(ui/…/bigtrace/index.ts::addBatteryStatsState) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack)

860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack) --> 37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(ui/…/bigtrace/index.ts::addTrack)

37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(ui/…/bigtrace/index.ts::addTrack) --> e8ce06d89b3a97cd61ee78b43fa487b166e0c86d4c988d8a3035da2ec78e88d1(ui/…/bigtrace/index.ts::getOrCreateGroup)

37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(ui/…/bigtrace/index.ts::addTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

e8ce06d89b3a97cd61ee78b43fa487b166e0c86d4c988d8a3035da2ec78e88d1(ui/…/bigtrace/index.ts::getOrCreateGroup) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query) --> c9d8b603589d22054ec5eb21b3ee6eed35eeab20f2b0edb56fac29bc392c8fa9(ui/…/bigtrace/index.ts::addBatteryStatsEvent)

c9d8b603589d22054ec5eb21b3ee6eed35eeab20f2b0edb56fac29bc392c8fa9(ui/…/bigtrace/index.ts::addBatteryStatsEvent) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack)

7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(ui/…/bigtrace/index.ts::addCounterTrack) --> 37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(ui/…/bigtrace/index.ts::addTrack)

75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(ui/…/bigtrace/index.ts::addDeviceState) --> c9d8b603589d22054ec5eb21b3ee6eed35eeab20f2b0edb56fac29bc392c8fa9(ui/…/bigtrace/index.ts::addBatteryStatsEvent)

75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(ui/…/bigtrace/index.ts::addDeviceState) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(ui/…/bigtrace/index.ts::addDeviceState) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack)

75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(ui/…/bigtrace/index.ts::addDeviceState) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(ui/…/bigtrace/index.ts::getOrCreateStandardGroup)

8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(ui/…/bigtrace/index.ts::getOrCreateStandardGroup) --> b1525680bca4aa5ed9ae20df97fa23019466ce899f29c1494f761773594c39fc(ui/…/public/workspace.ts::Workspace.addChildInOrder)

b1525680bca4aa5ed9ae20df97fa23019466ce899f29c1494f761773594c39fc(ui/…/public/workspace.ts::Workspace.addChildInOrder) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

bda3161d36863754d81d970e5e6d51afe41fcb86f9f05bdd65a8ba1cbbfd91d7(ui/…/bigtrace/index.ts::addHighCpu) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

bda3161d36863754d81d970e5e6d51afe41fcb86f9f05bdd65a8ba1cbbfd91d7(ui/…/bigtrace/index.ts::addHighCpu) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(ui/…/bigtrace/index.ts::addCounterTrack)

cbd75d0cd27c846937eb51087608a25cfb59af15b4cb872a4018155bee70c94f(ui/…/bigtrace/index.ts::addAtomCounters) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

cbd75d0cd27c846937eb51087608a25cfb59af15b4cb872a4018155bee70c94f(ui/…/bigtrace/index.ts::addAtomCounters) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(ui/…/bigtrace/index.ts::addCounterTrack)

8824ae128b64a8cfdcf585e73b0182936397a50ca6de49ba8b36a4f5823eb820(ui/…/bigtrace/index.ts::addAtomSlices) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

8824ae128b64a8cfdcf585e73b0182936397a50ca6de49ba8b36a4f5823eb820(ui/…/bigtrace/index.ts::addAtomSlices) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack)

931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(ui/…/bigtrace/index.ts::addModemMintData) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(ui/…/bigtrace/index.ts::addModemMintData) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack)

931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(ui/…/bigtrace/index.ts::addModemMintData) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(ui/…/bigtrace/index.ts::addCounterTrack)

b554d4e13f81a5711eabf94baf7204fb3d0384e10884576efefc6dcce8ec8c08(ui/…/bigtrace/index.ts::addKernelWakelocks) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

b554d4e13f81a5711eabf94baf7204fb3d0384e10884576efefc6dcce8ec8c08(ui/…/bigtrace/index.ts::addKernelWakelocks) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(ui/…/bigtrace/index.ts::addCounterTrack)

8c3f396c3b197b8a51586f0028985419207cf18e362e7a10476b559f9390f14a(ui/…/bigtrace/index.ts::addKernelWakelocksStatsd) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

8c3f396c3b197b8a51586f0028985419207cf18e362e7a10476b559f9390f14a(ui/…/bigtrace/index.ts::addKernelWakelocksStatsd) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(ui/…/bigtrace/index.ts::addCounterTrack)

35ece870e0bd27cf5ef389cba4be5d5cff4c191cbd8483c175f6edf423020991(ui/…/bigtrace/index.ts::addWakeups) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(ui/…/bigtrace/index.ts::query)

35ece870e0bd27cf5ef389cba4be5d5cff4c191cbd8483c175f6edf423020991(ui/…/bigtrace/index.ts::addWakeups) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(ui/…/bigtrace/index.ts::addSliceTrack)

01a10f3518909909ba091bd669f4ec3e20f7d9f23cd309e48a5067c2f9a4fe30(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.onTraceLoad) --> 8e8ab75932017412b46176380df1a23de7a76a5c0d0a308b0899b00d15402c63(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addCounters)

01a10f3518909909ba091bd669f4ec3e20f7d9f23cd309e48a5067c2f9a4fe30(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.onTraceLoad) --> 5964eaa985ab47e2a5a4f7e775f65b463f3181d29d9db3c18458003129c5baae(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addSlices)

8e8ab75932017412b46176380df1a23de7a76a5c0d0a308b0899b00d15402c63(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addCounters) --> e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addTrack)

e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addTrack) --> 37643660fca773aa0cae9360b78406a4e6ae917befb5ae6fd99945c21e143f9d(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.getGroupByName)

e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addTrack) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(ui/…/bigtrace/index.ts::getOrCreateStandardGroup)

37643660fca773aa0cae9360b78406a4e6ae917befb5ae6fd99945c21e143f9d(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.getGroupByName) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

5964eaa985ab47e2a5a4f7e775f65b463f3181d29d9db3c18458003129c5baae(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addSlices) --> e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(ui/…/bigtrace/index.ts::TraceProcessorTrackPlugin.addTrack)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> 4d1584ff18b4e64144aa065f8035490cd2e1eb1bfd01ea9a5b5a0bb6b80abbb0(ui/…/bigtrace/index.ts::addSliceTrackWithCustomColorizer)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> 38795aba13b2db87a7ece44e4fde8c2d1cabc717b76cf87cb2576f221463135a(ui/…/bigtrace/index.ts::addInstantTrack)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> 2a22ac7d78305a6d912a72e33b36f89c04cb1df0fd398ae1afca17e5e3e72c8b(ui/…/bigtrace/index.ts::addFlatSliceTrack)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> 308a2d3a5fbdbdc5c68105e700e6c5951ebd3b830c0cfb11b960adcc7fb444fa(ui/…/bigtrace/index.ts::addFixedColorSliceTrack)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> b554ddba8619afeac77afdf51cac31c729ce57066be526cbbc67a95ec7cf743a(ui/…/bigtrace/index.ts::addNestedTrackGroup)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> f9c692d2755bb8a002ed95e3f45480d638cdfa6e5075940b7ace09177b4b7e79(ui/…/bigtrace/index.ts::addTracksWithHelpText)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> e6334428616780de2e32b2601b9d53862275947b29d7ac6f4905a8f16db46774(ui/…/bigtrace/index.ts::addBasicSliceTrack)

8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(ui/…/bigtrace/index.ts::onTraceLoad) --> 7ed2c6efbcae89412449f63f4be7b229e233561e240884a893b71b2fe7b598df(ui/…/bigtrace/index.ts::addFilteredSliceTrack)

4d1584ff18b4e64144aa065f8035490cd2e1eb1bfd01ea9a5b5a0bb6b80abbb0(ui/…/bigtrace/index.ts::addSliceTrackWithCustomColorizer) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

38795aba13b2db87a7ece44e4fde8c2d1cabc717b76cf87cb2576f221463135a(ui/…/bigtrace/index.ts::addInstantTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

2a22ac7d78305a6d912a72e33b36f89c04cb1df0fd398ae1afca17e5e3e72c8b(ui/…/bigtrace/index.ts::addFlatSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

308a2d3a5fbdbdc5c68105e700e6c5951ebd3b830c0cfb11b960adcc7fb444fa(ui/…/bigtrace/index.ts::addFixedColorSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

b554ddba8619afeac77afdf51cac31c729ce57066be526cbbc67a95ec7cf743a(ui/…/bigtrace/index.ts::addNestedTrackGroup) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

f9c692d2755bb8a002ed95e3f45480d638cdfa6e5075940b7ace09177b4b7e79(ui/…/bigtrace/index.ts::addTracksWithHelpText) --> e0572e931267a5011088251093aa9b317154ef973ee05a789bccd6c2b0271128(ui/…/bigtrace/index.ts::addGroupWithHelpText)

e0572e931267a5011088251093aa9b317154ef973ee05a789bccd6c2b0271128(ui/…/bigtrace/index.ts::addGroupWithHelpText) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

e6334428616780de2e32b2601b9d53862275947b29d7ac6f4905a8f16db46774(ui/…/bigtrace/index.ts::addBasicSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

7ed2c6efbcae89412449f63f4be7b229e233561e240884a893b71b2fe7b598df(ui/…/bigtrace/index.ts::addFilteredSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(ui/…/bigtrace/index.ts::onTraceLoad) --> 7c65bba5da65ce25b4375fb3bfda9bd157959654eec670c790efc1c96cbe5871(ui/…/bigtrace/index.ts::addScrollJankV3ScrollTrack)

56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(ui/…/bigtrace/index.ts::onTraceLoad) --> 57922b1efe204c210f57ce76be72c7bf970d51ccde3ab1ae37a39f0e92512ac3(ui/…/bigtrace/index.ts::addScrollTimelineTrack)

56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(ui/…/bigtrace/index.ts::onTraceLoad) --> d8c5d347b56b2e1e7c34b64c4bfb740cf75b63cebd997aba6bdbede885ed0c7f(ui/…/bigtrace/index.ts::addVsyncTracks)

56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(ui/…/bigtrace/index.ts::onTraceLoad) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(ui/…/bigtrace/index.ts::onTraceLoad) --> d9c6eda7e8985804e9c1ffc0610a944531012ae7222b5055e41641165e234286(ui/…/bigtrace/index.ts::addTopLevelScrollTrack)

56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(ui/…/bigtrace/index.ts::onTraceLoad) --> 3d39b0d2c6f6e101bea27a66aa7e42a2fa0614258e5297ce12bf3e5283133a0b(ui/…/bigtrace/index.ts::addEventLatencyTrack)

7c65bba5da65ce25b4375fb3bfda9bd157959654eec670c790efc1c96cbe5871(ui/…/bigtrace/index.ts::addScrollJankV3ScrollTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

57922b1efe204c210f57ce76be72c7bf970d51ccde3ab1ae37a39f0e92512ac3(ui/…/bigtrace/index.ts::addScrollTimelineTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

d8c5d347b56b2e1e7c34b64c4bfb740cf75b63cebd997aba6bdbede885ed0c7f(ui/…/bigtrace/index.ts::addVsyncTracks) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

d9c6eda7e8985804e9c1ffc0610a944531012ae7222b5055e41641165e234286(ui/…/bigtrace/index.ts::addTopLevelScrollTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

3d39b0d2c6f6e101bea27a66aa7e42a2fa0614258e5297ce12bf3e5283133a0b(ui/…/bigtrace/index.ts::addEventLatencyTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(ui/…/bigtrace/index.ts::onTraceLoad) --> 14aa6d26f196fb6ec83b341f7574402cbec3941a8e68faad0834faa792bfde57(ui/…/bigtrace/index.ts::addGlobalAllocs)

a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(ui/…/bigtrace/index.ts::onTraceLoad) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(ui/…/bigtrace/index.ts::onTraceLoad) --> 3ca39ac61a2ce22ede1419811aa0699435f6e788359b095d47e5f3ac08f94ce0(ui/…/bigtrace/index.ts::addGlobalCounter)

a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(ui/…/bigtrace/index.ts::onTraceLoad) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(ui/…/bigtrace/index.ts::getOrCreateStandardGroup)

a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(ui/…/bigtrace/index.ts::onTraceLoad) --> 810899c2ef4ce611a25d20f16c2278963f13f5cec1c0186fb7ec5f7affb0e24c(ui/…/bigtrace/index.ts::memoryGroupFn)

14aa6d26f196fb6ec83b341f7574402cbec3941a8e68faad0834faa792bfde57(ui/…/bigtrace/index.ts::addGlobalAllocs) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

3ca39ac61a2ce22ede1419811aa0699435f6e788359b095d47e5f3ac08f94ce0(ui/…/bigtrace/index.ts::addGlobalCounter) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(ui/…/public/workspace.ts::TrackNode.addChildInOrder)

810899c2ef4ce611a25d20f16c2278963f13f5cec1c0186fb7ec5f7affb0e24c(ui/…/bigtrace/index.ts::memoryGroupFn) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(ui/…/bigtrace/index.ts::getOrCreateStandardGroup)


classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9

%% Swimm:
%% graph TD;
%%       1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addNetworkSummary)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addDeviceState)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> bda3161d36863754d81d970e5e6d51afe41fcb86f9f05bdd65a8ba1cbbfd91d7(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addHighCpu)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> cbd75d0cd27c846937eb51087608a25cfb59af15b4cb872a4018155bee70c94f(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addAtomCounters)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 8824ae128b64a8cfdcf585e73b0182936397a50ca6de49ba8b36a4f5823eb820(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addAtomSlices)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addModemMintData)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> b554d4e13f81a5711eabf94baf7204fb3d0384e10884576efefc6dcce8ec8c08(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addKernelWakelocks)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 8c3f396c3b197b8a51586f0028985419207cf18e362e7a10476b559f9390f14a(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addKernelWakelocksStatsd)
%% 
%% 1c4b6a78549280107034cea525cff78ade61bd1cdf22c04fa4543c5396780076(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 35ece870e0bd27cf5ef389cba4be5d5cff4c191cbd8483c175f6edf423020991(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addWakeups)
%% 
%% 56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addNetworkSummary) --> e2e80e4b100e8dcc4fb98fd036ee752172fa1be983fea9645909304414f45dd6(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addBatteryStatsState)
%% 
%% 56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addNetworkSummary) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% 56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addNetworkSummary) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack)
%% 
%% 56bd48e062a99877d63ed2c4dd8f3ba0f4e77402d2f5f2468bb8cec3ab994f84(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addNetworkSummary) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addCounterTrack)
%% 
%% e2e80e4b100e8dcc4fb98fd036ee752172fa1be983fea9645909304414f45dd6(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addBatteryStatsState) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack)
%% 
%% 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack) --> 37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTrack)
%% 
%% 37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTrack) --> e8ce06d89b3a97cd61ee78b43fa487b166e0c86d4c988d8a3035da2ec78e88d1(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::getOrCreateGroup)
%% 
%% 37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% e8ce06d89b3a97cd61ee78b43fa487b166e0c86d4c988d8a3035da2ec78e88d1(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::getOrCreateGroup) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query) --> c9d8b603589d22054ec5eb21b3ee6eed35eeab20f2b0edb56fac29bc392c8fa9(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addBatteryStatsEvent)
%% 
%% c9d8b603589d22054ec5eb21b3ee6eed35eeab20f2b0edb56fac29bc392c8fa9(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addBatteryStatsEvent) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack)
%% 
%% 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addCounterTrack) --> 37f40643ef35a7c8e225e630c7bb18b695ef81f0358c764e7db6c44ad82df345(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTrack)
%% 
%% 75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addDeviceState) --> c9d8b603589d22054ec5eb21b3ee6eed35eeab20f2b0edb56fac29bc392c8fa9(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addBatteryStatsEvent)
%% 
%% 75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addDeviceState) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% 75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addDeviceState) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack)
%% 
%% 75bde2c82b26f2d46e5c7520e491ec71b2f086a21042228040c344d9b696cb36(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addDeviceState) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::getOrCreateStandardGroup)
%% 
%% 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::getOrCreateStandardGroup) --> b1525680bca4aa5ed9ae20df97fa23019466ce899f29c1494f761773594c39fc(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::Workspace.addChildInOrder)
%% 
%% b1525680bca4aa5ed9ae20df97fa23019466ce899f29c1494f761773594c39fc(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::Workspace.addChildInOrder) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% bda3161d36863754d81d970e5e6d51afe41fcb86f9f05bdd65a8ba1cbbfd91d7(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addHighCpu) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% bda3161d36863754d81d970e5e6d51afe41fcb86f9f05bdd65a8ba1cbbfd91d7(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addHighCpu) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addCounterTrack)
%% 
%% cbd75d0cd27c846937eb51087608a25cfb59af15b4cb872a4018155bee70c94f(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addAtomCounters) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% cbd75d0cd27c846937eb51087608a25cfb59af15b4cb872a4018155bee70c94f(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addAtomCounters) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addCounterTrack)
%% 
%% 8824ae128b64a8cfdcf585e73b0182936397a50ca6de49ba8b36a4f5823eb820(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addAtomSlices) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% 8824ae128b64a8cfdcf585e73b0182936397a50ca6de49ba8b36a4f5823eb820(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addAtomSlices) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack)
%% 
%% 931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addModemMintData) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% 931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addModemMintData) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack)
%% 
%% 931a95edf64a2a94205ad333a54f81aba02a72642916c6c8eb4c2f916f76c381(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addModemMintData) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addCounterTrack)
%% 
%% b554d4e13f81a5711eabf94baf7204fb3d0384e10884576efefc6dcce8ec8c08(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addKernelWakelocks) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% b554d4e13f81a5711eabf94baf7204fb3d0384e10884576efefc6dcce8ec8c08(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addKernelWakelocks) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addCounterTrack)
%% 
%% 8c3f396c3b197b8a51586f0028985419207cf18e362e7a10476b559f9390f14a(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addKernelWakelocksStatsd) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% 8c3f396c3b197b8a51586f0028985419207cf18e362e7a10476b559f9390f14a(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addKernelWakelocksStatsd) --> 7d6c8c57bf35157528e7aa7c52365b6b6509d026dd2c8b9d9d2bbc259ca6f697(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addCounterTrack)
%% 
%% 35ece870e0bd27cf5ef389cba4be5d5cff4c191cbd8483c175f6edf423020991(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addWakeups) --> 62025d91f6da47d8c1da3d01eb587e78fa2b5c1b91c4b322612d837f9b78852b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::query)
%% 
%% 35ece870e0bd27cf5ef389cba4be5d5cff4c191cbd8483c175f6edf423020991(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addWakeups) --> 860b1baa35c1a4772bdc62a157f94b4820a2343a95822efb7b6951bf3de521e3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrack)
%% 
%% 01a10f3518909909ba091bd669f4ec3e20f7d9f23cd309e48a5067c2f9a4fe30(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.onTraceLoad) --> 8e8ab75932017412b46176380df1a23de7a76a5c0d0a308b0899b00d15402c63(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addCounters)
%% 
%% 01a10f3518909909ba091bd669f4ec3e20f7d9f23cd309e48a5067c2f9a4fe30(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.onTraceLoad) --> 5964eaa985ab47e2a5a4f7e775f65b463f3181d29d9db3c18458003129c5baae(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addSlices)
%% 
%% 8e8ab75932017412b46176380df1a23de7a76a5c0d0a308b0899b00d15402c63(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addCounters) --> e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addTrack)
%% 
%% e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addTrack) --> 37643660fca773aa0cae9360b78406a4e6ae917befb5ae6fd99945c21e143f9d(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.getGroupByName)
%% 
%% e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addTrack) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::getOrCreateStandardGroup)
%% 
%% 37643660fca773aa0cae9360b78406a4e6ae917befb5ae6fd99945c21e143f9d(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.getGroupByName) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 5964eaa985ab47e2a5a4f7e775f65b463f3181d29d9db3c18458003129c5baae(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addSlices) --> e077f6e02c8c6b24435bc839b5fb7c158abc3ac9b0f81d3841aed68812b1128d(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::TraceProcessorTrackPlugin.addTrack)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 4d1584ff18b4e64144aa065f8035490cd2e1eb1bfd01ea9a5b5a0bb6b80abbb0(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrackWithCustomColorizer)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 38795aba13b2db87a7ece44e4fde8c2d1cabc717b76cf87cb2576f221463135a(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addInstantTrack)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 2a22ac7d78305a6d912a72e33b36f89c04cb1df0fd398ae1afca17e5e3e72c8b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addFlatSliceTrack)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 308a2d3a5fbdbdc5c68105e700e6c5951ebd3b830c0cfb11b960adcc7fb444fa(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addFixedColorSliceTrack)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> b554ddba8619afeac77afdf51cac31c729ce57066be526cbbc67a95ec7cf743a(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addNestedTrackGroup)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> f9c692d2755bb8a002ed95e3f45480d638cdfa6e5075940b7ace09177b4b7e79(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTracksWithHelpText)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> e6334428616780de2e32b2601b9d53862275947b29d7ac6f4905a8f16db46774(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addBasicSliceTrack)
%% 
%% 8886a730e844277bb56c98efe0a36425b0b48bf5d51742f03e98df62c5c27a54(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 7ed2c6efbcae89412449f63f4be7b229e233561e240884a893b71b2fe7b598df(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addFilteredSliceTrack)
%% 
%% 4d1584ff18b4e64144aa065f8035490cd2e1eb1bfd01ea9a5b5a0bb6b80abbb0(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addSliceTrackWithCustomColorizer) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 38795aba13b2db87a7ece44e4fde8c2d1cabc717b76cf87cb2576f221463135a(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addInstantTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 2a22ac7d78305a6d912a72e33b36f89c04cb1df0fd398ae1afca17e5e3e72c8b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addFlatSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 308a2d3a5fbdbdc5c68105e700e6c5951ebd3b830c0cfb11b960adcc7fb444fa(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addFixedColorSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% b554ddba8619afeac77afdf51cac31c729ce57066be526cbbc67a95ec7cf743a(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addNestedTrackGroup) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% f9c692d2755bb8a002ed95e3f45480d638cdfa6e5075940b7ace09177b4b7e79(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTracksWithHelpText) --> e0572e931267a5011088251093aa9b317154ef973ee05a789bccd6c2b0271128(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addGroupWithHelpText)
%% 
%% e0572e931267a5011088251093aa9b317154ef973ee05a789bccd6c2b0271128(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addGroupWithHelpText) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% e6334428616780de2e32b2601b9d53862275947b29d7ac6f4905a8f16db46774(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addBasicSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 7ed2c6efbcae89412449f63f4be7b229e233561e240884a893b71b2fe7b598df(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addFilteredSliceTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 7c65bba5da65ce25b4375fb3bfda9bd157959654eec670c790efc1c96cbe5871(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addScrollJankV3ScrollTrack)
%% 
%% 56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 57922b1efe204c210f57ce76be72c7bf970d51ccde3ab1ae37a39f0e92512ac3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addScrollTimelineTrack)
%% 
%% 56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> d8c5d347b56b2e1e7c34b64c4bfb740cf75b63cebd997aba6bdbede885ed0c7f(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addVsyncTracks)
%% 
%% 56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> d9c6eda7e8985804e9c1ffc0610a944531012ae7222b5055e41641165e234286(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTopLevelScrollTrack)
%% 
%% 56998705e7a6fa230a8245393e673fb9abeedae7e63c6c8e1d7550c612ac0985(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 3d39b0d2c6f6e101bea27a66aa7e42a2fa0614258e5297ce12bf3e5283133a0b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addEventLatencyTrack)
%% 
%% 7c65bba5da65ce25b4375fb3bfda9bd157959654eec670c790efc1c96cbe5871(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addScrollJankV3ScrollTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 57922b1efe204c210f57ce76be72c7bf970d51ccde3ab1ae37a39f0e92512ac3(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addScrollTimelineTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% d8c5d347b56b2e1e7c34b64c4bfb740cf75b63cebd997aba6bdbede885ed0c7f(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addVsyncTracks) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% d9c6eda7e8985804e9c1ffc0610a944531012ae7222b5055e41641165e234286(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addTopLevelScrollTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 3d39b0d2c6f6e101bea27a66aa7e42a2fa0614258e5297ce12bf3e5283133a0b(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addEventLatencyTrack) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 14aa6d26f196fb6ec83b341f7574402cbec3941a8e68faad0834faa792bfde57(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addGlobalAllocs)
%% 
%% a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 3ca39ac61a2ce22ede1419811aa0699435f6e788359b095d47e5f3ac08f94ce0(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addGlobalCounter)
%% 
%% a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::getOrCreateStandardGroup)
%% 
%% a9863466799f3b767b42c113c829fdff5b709cc99145f64b1e9edb7f1202c406(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::onTraceLoad) --> 810899c2ef4ce611a25d20f16c2278963f13f5cec1c0186fb7ec5f7affb0e24c(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::memoryGroupFn)
%% 
%% 14aa6d26f196fb6ec83b341f7574402cbec3941a8e68faad0834faa792bfde57(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addGlobalAllocs) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 3ca39ac61a2ce22ede1419811aa0699435f6e788359b095d47e5f3ac08f94ce0(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::addGlobalCounter) --> 662c2de27c08a95352870e754e1224b956982a2f87bf061651e17b8bd69da718(<SwmPath>[ui/…/public/workspace.ts](ui/src/public/workspace.ts)</SwmPath>::TrackNode.addChildInOrder)
%% 
%% 810899c2ef4ce611a25d20f16c2278963f13f5cec1c0186fb7ec5f7affb0e24c(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::memoryGroupFn) --> 8249ce7aed1eed746bcdf430afb682d23a737348091f95376fc077054726a3c8(<SwmPath>[ui/…/bigtrace/index.ts](ui/src/bigtrace/index.ts)</SwmPath>::getOrCreateStandardGroup)
%% 
%% 
%% classDef mainFlowStyle color:#000000,fill:#7CB9F4
%% classDef rootsStyle color:#000000,fill:#00FFF4
%% classDef Style1 color:#000000,fill:#00FFAA
%% classDef Style2 color:#000000,fill:#FFFF00
%% classDef Style3 color:#000000,fill:#AA7CB9
```

# Ordering and Inserting Child Nodes

<SwmSnippet path="/ui/src/public/workspace.ts" line="330">

---

AddChildInOrder kicks off the process by looking for the first child whose <SwmToken path="ui/src/public/workspace.ts" pos="332:10:10" line-data="      (n) =&gt; (n.sortOrder ?? 0) &gt; (child.sortOrder ?? 0),">`sortOrder`</SwmToken> is greater than the incoming child's <SwmToken path="ui/src/public/workspace.ts" pos="332:10:10" line-data="      (n) =&gt; (n.sortOrder ?? 0) &gt; (child.sortOrder ?? 0),">`sortOrder`</SwmToken>. If it finds one, it hands off to <SwmToken path="ui/src/public/workspace.ts" pos="335:5:5" line-data="      return this.addChildBefore(child, insertPoint);">`addChildBefore`</SwmToken> to do the actual insertion, making sure the new child lands in the right spot. If not, it just appends the child at the end. This keeps the ordering logic clean and avoids duplicating how nodes are inserted.

```typescript
  addChildInOrder(child: TrackNode): Result {
    const insertPoint = this._children.find(
      (n) => (n.sortOrder ?? 0) > (child.sortOrder ?? 0),
    );
    if (insertPoint) {
      return this.addChildBefore(child, insertPoint);
    } else {
      return this.addChildLast(child);
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/public/workspace.ts" line="372">

---

AddChildBefore handles the actual insertion by first making sure we're not trying to insert a node before itself, then checks that the <SwmToken path="ui/src/public/workspace.ts" pos="372:9:9" line-data="  addChildBefore(child: TrackNode, referenceNode: TrackNode): Result {">`referenceNode`</SwmToken> is actually a child. It calls adopt to set up the parent-child relationship, and if that's good, it splices the new child in right before the <SwmToken path="ui/src/public/workspace.ts" pos="372:9:9" line-data="  addChildBefore(child: TrackNode, referenceNode: TrackNode): Result {">`referenceNode`</SwmToken>. This keeps the tree structure and ordering intact.

```typescript
  addChildBefore(child: TrackNode, referenceNode: TrackNode): Result {
    // Nodes are the same, nothing to do.
    if (child === referenceNode) return okResult();

    assertTrue(this.children.includes(referenceNode));

    const result = this.adopt(child);
    if (!result.ok) return result;

    const indexOfReference = this.children.indexOf(referenceNode);
    this._children.splice(indexOfReference, 0, child);

    return okResult();
  }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
