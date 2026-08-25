# Cruce IDs Flash (G4V) ↔ Game Server MexOrbit

Cruce de los **582** IDs del SWF (`Decompile/main/.../§_-G4V§`) con el código en `MexOrbit.GameServer/Net/netty/`: carpetas **commands** (paquetes servidor→cliente), **requests** (cliente→servidor) y rutas registradas en **Handler.cs**.

## Resumen

| Métrica | Valor |
|--------|------:|
| IDs en el cliente Flash | 582 |
| IDs con implementación o handler en MexOrbit | 204 |
| Sin correspondencia en este servidor | 378 |

## Lectura rápida

- **server->client:** clase en `netty/commands` — el servidor serializa y envía al Flash.
- **client->server:** clase `*Request` en `netty/requests`, o fila *(Handler.cs)* si no hay archivo Request dedicado.
- Varios nombres para un mismo ID indican submódulos o duplicados de ID en el protocolo original.

## Tabla ID → Flash → MexOrbit

| ID | Archivo Flash | MexOrbit |
|----:|---------------|----------|
| 39 | `§_-z3n§.as` | — |
| 52 | `§_-41i§.as` | server->client: MineCreateCommand |
| 65 | `§_-c4E§.as` | client->server: UbaMatchmakingAcceptRequestHandler (Handler.cs) |
| 71 | `§_-e3m§.as` | — |
| 368 | `§_-T24§.as` | — |
| 423 | `§_-w1Z§.as` | server->client: UserKeyBindingsModule |
| 532 | `§_-i4Q§.as` | — |
| 540 | `§_-Y3z§.as` | client->server: GroupChangeLeaderRequest |
| 596 | `§_-I3d§.as` | server->client: GroupInviteCommand |
| 632 | `§_-A30§.as` | — |
| 645 | `§_-nm§.as` | client->server: PetRequest |
| 659 | `§_-mM§.as` | client->server: UbaMatchmakingCancelRequestHandler (Handler.cs) |
| 666 | `§_-36§.as` | — |
| 667 | `§_-k3l§.as` | — |
| 686 | `§_-E4x§.as` | server->client: ClientUISlotBarCategoryItemTimerStateModule |
| 702 | `§_-K18§.as` | server->client: class_K18 |
| 758 | `§_-91S§.as` | client->server: GroupFollowPlayerRequest |
| 770 | `§_-X4J§.as` | server->client: BattleStationBuildingUiInitializationCommand |
| 784 | `§_-24C§.as` | — |
| 802 | `§_-a2N§_2.as` | — |
| 1038 | `§_-W2f§.as` | — |
| 1059 | `§_-w1b§.as` | server->client: ClientUITextReplacementModule<br>server->client: MessageWildcardReplacementModule |
| 1084 | `§_-t3b§.as` | — |
| 1096 | `§_-l4D§.as` | — |
| 1110 | `§_-tU§.as` | server->client: WindowSettingsModule |
| 1116 | `§_-53v§.as` | — |
| 1178 | `§_-X2C§.as` | — |
| 1271 | `§_-Q1a§.as` | server->client: VideoWindowCreateCommand |
| 1284 | `§_-539§.as` | — |
| 1338 | `§_-l3f§.as` | — |
| 1421 | `§_-L29§.as` | server->client: AvailableModulesCommand |
| 1470 | `§_-L2Q§.as` | — |
| 1528 | `§_-C2g§.as` | client->server: AttackAbortLaserRequestHandler (Handler.cs) |
| 1885 | `§_-k4a§.as` | — |
| 1939 | `§_-P1e§.as` | — |
| 1992 | `§_-14T§.as` | — |
| 1995 | `§_-d4r§.as` | server->client: AsteroidProgressCommand |
| 2015 | `§_-84I§.as` | server->client: class_84I |
| 2068 | `§_-cN§.as` | — |
| 2072 | `§_-T4o§.as` | server->client: UpdateMenuItemCooldownGroupTimerCommand |
| 2083 | `§_-X1X§.as` | — |
| 2138 | `§_-53n§.as` | — |
| 2150 | `§_-r3f§.as` | server->client: GroupPlayerClanModule |
| 2157 | `§_-I4H§_2.as` | — |
| 2244 | `§_-u2k§.as` | client->server: RepairStationRequestHandler (Handler.cs) |
| 2289 | `§_-w2N§.as` | server->client: FactionModule |
| 2310 | `§_-L3W§.as` | server->client: ActivatePortalCommand |
| 2311 | `§_-E1Q§.as` | — |
| 2357 | `§_-64h§.as` | — |
| 2376 | `§_-D2x§.as` | — |
| 2419 | `§_-t1O§.as` | — |
| 2431 | `§_-S1w§.as` | — |
| 2496 | `§_-c1O§.as` | server->client: GroupInitializationCommand |
| 2520 | `§_-z2§.as` | client->server: EquipModuleRequest |
| 2586 | `§_-F3m§.as` | server->client: AbilityStopCommand |
| 2587 | `§_-U4H§.as` | — |
| 2599 | `§_-Td§.as` | — |
| 2633 | `§_-r35§.as` | — |
| 2643 | `§_-uO§.as` | server->client: AbilityEffectActivationCommand |
| 2646 | `§_-c37§.as` | — |
| 2656 | `§_-i2N§.as` | client->server: GroupPingPositionRequest |
| 2958 | `§_-j2v§.as` | — |
| 2978 | `§_-Qr§_2.as` | server->client: GroupRemoveInvitationCommand |
| 3062 | `§_-n2§.as` | server->client: BattleStationNoClanUiInitializationCommand |
| 3164 | `§_-Zs§.as` | — |
| 3186 | `§_-a3Z§.as` | — |
| 3220 | `§_-i47§.as` | — |
| 3248 | `§_-P2a§.as` | — |
| 3372 | `§_-I1W§.as` | server->client: class_I1W |
| 3383 | `§_-J39§.as` | server->client: UbaJ39Module |
| 3388 | `§_-p2i§.as` | — |
| 3397 | `§_-S46§.as` | server->client: AssetRemoveCommand |
| 3424 | `§_-e3L§.as` | — |
| 3561 | `§_-a1o§.as` | — |
| 3574 | `§_-m2Q§.as` | server->client: JumpCPUPriceMappingModule |
| 3612 | `§_-J5§.as` | — |
| 3734 | `§_-g3s§.as` | — |
| 3758 | `§_-pI§.as` | server->client: AttackHitCommand |
| 3801 | `§_-NH§.as` | — |
| 3826 | `§_-b3x§.as` | — |
| 3850 | `§_-R3G§.as` | — |
| 3946 | `§_-g1a§.as` | server->client: class_g1a |
| 4013 | `§_-C2u§.as` | — |
| 4062 | `§_-IC§.as` | server->client: AttributeSkillShieldUpdateCommand |
| 4082 | `§_-O2S§.as` | server->client: SetSpeedCommand |
| 4108 | `§_-53O§.as` | — |
| 4159 | `§_-H4Z§.as` | server->client: DroneFormationChangeCommand |
| 4182 | `§_-L3B§.as` | — |
| 4224 | `§_-Q1n§.as` | server->client: LegacyModule<br>client->server: LegacyModuleRequest |
| 4246 | `§_-L17§.as` | — |
| 4330 | `§_-11C§.as` | server->client: PetUIRepairButtonCommand |
| 4349 | `§_-G3D§.as` | — |
| 4361 | `§_-m31§.as` | — |
| 4390 | `§_-Ij§_2.as` | — |
| 4423 | `§_-XZ§.as` | — |
| 4439 | `§_-L45§.as` | — |
| 4444 | `§_-f2d§.as` | — |
| 4479 | `§_-A2n§.as` | — |
| 4530 | `§_-94W§.as` | — |
| 4550 | `§_-k34§.as` | server->client: AttributeShieldUpdateCommand |
| 4585 | `§_-b27§.as` | — |
| 4609 | `§_-W3v§.as` | — |
| 4738 | `§_-E2Y§.as` | — |
| 4853 | `§_-SC§.as` | — |
| 4873 | `§_-p2w§.as` | server->client: BattleStationBuildingStateCommand |
| 5029 | `§_-F4I§.as` | — |
| 5107 | `§_-d3E§.as` | — |
| 5174 | `§_-p2C§.as` | — |
| 5388 | `§_-NT§.as` | — |
| 5452 | `§_-W17§.as` | — |
| 5475 | `§_-xX§.as` | — |
| 5503 | `§_-D4Y§.as` | — |
| 5597 | `§_-B2P§.as` | — |
| 5599 | `§_-54u§.as` | — |
| 5610 | `§_-83S§.as` | — |
| 5614 | `§_-h5§.as` | — |
| 5744 | `§_-63t§.as` | — |
| 5748 | `§_-m2Y§.as` | — |
| 5772 | `§_-Q2d§.as` | — |
| 5776 | `§_-hS§.as` | server->client: UbahsModule |
| 5862 | `§_-hh§.as` | — |
| 5888 | `§_-A2e§.as` | — |
| 6089 | `§_-M3P§.as` | — |
| 6195 | `§_-e4z§.as` | server->client: BoosterUpdateModule |
| 6275 | `§_-t2i§.as` | — |
| 6518 | `§_-fE§.as` | server->client: AttributeBoosterUpdateCommand |
| 6532 | `§_-cg§.as` | client->server: CollectBoxRequest |
| 6542 | `§_-NQ§.as` | — |
| 6619 | `§_-83c§.as` | — |
| 6838 | `§_-k2a§.as` | — |
| 6949 | `§_-T1s§.as` | server->client: EquipReadyCommand |
| 6953 | `§_-f42§.as` | — |
| 7031 | `§_-F1L§.as` | — |
| 7044 | `§_-332§.as` | client->server: BuildStationRequest |
| 7101 | `§_-t2W§.as` | client->server: GroupRejectInvitationRequest |
| 7114 | `§_-346§.as` | — |
| 7174 | `§_-Y4F§.as` | — |
| 7240 | `§_-O2§.as` | server->client: MapAddPOICommand |
| 7270 | `§_-K2V§.as` | server->client: ShipCreateCommand |
| 7508 | `§_-S4w§.as` | — |
| 7511 | `§_-r1h§.as` | server->client: ShipInitializationCommand |
| 7606 | `§_-X28§.as` | client->server: SendWindowUpdateRequest |
| 7633 | `§_-QB§.as` | — |
| 7703 | `§_-S4Z§.as` | — |
| 7717 | `§_-047§.as` | server->client: Uba047Module |
| 7725 | `§_-e4W§.as` | server->client: command_e4W |
| 7758 | `§_-qN§.as` | — |
| 7765 | `§_-L3O§.as` | — |
| 7775 | `§_-hb§.as` | — |
| 7786 | `§_-g1G§.as` | server->client: CooldownTypeModule |
| 7787 | `§_-R4B§.as` | — |
| 7919 | `§_-h6§.as` | — |
| 7953 | `§_-J2M§.as` | — |
| 8012 | `§_-R4Y§.as` | — |
| 8020 | `§_-Z3J§.as` | client->server: DisplaySettingsRequest |
| 8188 | `§_-uk§.as` | — |
| 8216 | `§_-W2i§.as` | server->client: PetExperiencePointsUpdateCommand |
| 8290 | `§_-43G§.as` | server->client: Uba43GModule |
| 8401 | `§_-g4g§.as` | — |
| 8422 | `§_-t28§.as` | — |
| 8437 | `§_-tW§.as` | server->client: AssetInfoCommand |
| 8468 | `§_-Q1s§.as` | server->client: StationModuleModule |
| 8492 | `§_-82H§.as` | — |
| 8537 | `§_-D2s§.as` | — |
| 8578 | `§_-W4n§.as` | — |
| 8689 | `§_-01y§.as` | server->client: ContactsListUpdateCommand |
| 8793 | `§_-U4b§.as` | — |
| 9048 | `§_-i1t§.as` | — |
| 9103 | `§_-u26§.as` | client->server: GroupLeaveRequestHandler (Handler.cs) |
| 9151 | `§_-I3U§.as` | — |
| 9161 | `§_-72Y§.as` | — |
| 9174 | `§_-s35§.as` | server->client: PetInitializationCommand |
| 9181 | `§_-I3N§.as` | — |
| 9191 | `§_-z3C§.as` | — |
| 9253 | `§_-p2N§.as` | client->server: PortalJumpRequestHandler (Handler.cs) |
| 9286 | `§_-V2d§.as` | — |
| 9360 | `§_-C3x§.as` | server->client: AttackMissedCommand |
| 9450 | `§_-92X§.as` | — |
| 9460 | `§_-l2V§.as` | — |
| 9461 | `§_-X26§.as` | — |
| 9493 | `§_-93A§.as` | — |
| 9515 | `§_-DN§.as` | — |
| 9520 | `§_-S1g§.as` | — |
| 9702 | `§_-Q2x§.as` | — |
| 9855 | `§_-M4§.as` | — |
| 9939 | `§_-O1e§.as` | — |
| 10006 | `§_-Q1M§.as` | — |
| 10020 | `§_-92k§.as` | — |
| 10084 | `§_-H2I§.as` | server->client: BeaconCommand |
| 10114 | `§_-k4H§.as` | server->client: KillScreenPostCommand |
| 10196 | `§_-h2R§.as` | client->server: GroupRevokeInvitationRequest |
| 10343 | `§_-L1g§.as` | client->server: LogoutCancelRequestHandler (Handler.cs) |
| 10470 | `§_-R25§.as` | — |
| 10487 | `§_-o1J§.as` | server->client: PriceModule |
| 10530 | `§_-s2E§.as` | — |
| 10575 | `§_-Q4C§.as` | — |
| 10720 | `§_-y3i§.as` | server->client: class_y3i |
| 10794 | `§_-uZ§.as` | — |
| 10809 | `§_-HT§_2.as` | — |
| 10838 | `§_-L3q§.as` | — |
| 10905 | `§_-Q2K§.as` | — |
| 10939 | `§_-j1O§.as` | — |
| 10951 | `§_-x20§.as` | — |
| 10996 | `§_-V3G§.as` | client->server: LoginRequest |
| 11118 | `§_-h45§.as` | server->client: class_h45 |
| 11127 | `§_-84f§.as` | — |
| 11152 | `§_-R1v§.as` | — |
| 11175 | `§_-b2f§.as` | client->server: GroupAcceptInvitationRequest |
| 11246 | `§_-E1E§.as` | server->client: ClientUITooltipsCommand |
| 11252 | `§_-r39§.as` | — |
| 11298 | `§_-i1d§.as` | server->client: class_i1d |
| 11301 | `§_-S4d§.as` | client->server: UbaMatchmakingRequestHandler (Handler.cs) |
| 11406 | `§_-5§.as` | — |
| 11416 | `§_-V1C§.as` | — |
| 11468 | `§_-bG§_2.as` | server->client: ShipSelectionCommand |
| 11604 | `§_-c33§.as` | — |
| 11623 | `§_-j4q§.as` | server->client: ParseFeaturesMenuData |
| 11653 | `§_-2z§.as` | server->client: KillScreenOptionTypeModule |
| 11695 | `§_-U49§.as` | — |
| 11744 | `§_-l4b§.as` | server->client: Ubal4bModule |
| 11751 | `§_-71k§.as` | server->client: MessageLocalizedWildcardCommand |
| 11900 | `§_-R1K§.as` | — |
| 11917 | `§_-R2d§.as` | — |
| 11952 | `§_-C1j§.as` | — |
| 11985 | `§_-K3u§.as` | — |
| 12067 | `§_-Q4E§.as` | — |
| 12173 | `§_-j3s§.as` | — |
| 12207 | `§_-a2R§.as` | — |
| 12258 | `§_-61J§.as` | — |
| 12304 | `§_-L3e§.as` | — |
| 12308 | `§_-524§.as` | — |
| 12315 | `§_-i4c§.as` | — |
| 12351 | `§_-f3T§.as` | — |
| 12479 | `§_-H3o§.as` | — |
| 12560 | `§_-Q4G§.as` | — |
| 12647 | `§_-N3I§.as` | server->client: VisualModifierCommand |
| 12658 | `§_-d27§.as` | — |
| 12754 | `§_-a3G§.as` | server->client: ClientUISlotBarCategoryItemStatusModule |
| 12847 | `§_-n3d§.as` | — |
| 12879 | `§_-n3Q§.as` | — |
| 12908 | `§_-P3Z§.as` | — |
| 13016 | `§_-S3G§.as` | server->client: UserKeyBindingsUpdateCommand<br>client->server: UserKeyBindingsUpdateRequest |
| 13039 | `§_-73e§.as` | — |
| 13136 | `§_-21I§.as` | — |
| 13201 | `§_-64L§.as` | — |
| 13276 | `§_-C2E§.as` | — |
| 13359 | `§_-X1t§.as` | client->server: GroupInvitationRequest |
| 13473 | `§_-l3a§.as` | — |
| 13518 | `§_-m3k§.as` | — |
| 13538 | `§_-X2z§.as` | — |
| 13569 | `§_-Q4Z§.as` | — |
| 13691 | `§_-64i§.as` | server->client: Uba64iModule |
| 13718 | `§_-Uo§_2.as` | server->client: AssetCreateCommand |
| 13868 | `§_-Am§.as` | — |
| 13918 | `§_-O1w§.as` | — |
| 14030 | `§_-11d§.as` | server->client: class_11d |
| 14175 | `§_-d2K§.as` | server->client: GroupPlayerLeaveCommand |
| 14177 | `§_-M1M§.as` | — |
| 14222 | `§_-J24§.as` | server->client: AddMenuItemHighlightCommand |
| 14282 | `§_-83a§.as` | — |
| 14317 | `§_-g2l§.as` | server->client: UpdateWindowItemCommand |
| 14350 | `§_-41T§.as` | server->client: EquippedModulesModule |
| 14405 | `§_-Y3d§_2.as` | — |
| 14534 | `§_-H37§.as` | — |
| 14575 | `§_-24l§.as` | server->client: GroupUpdateUICommand |
| 14585 | `§_-s3w§.as` | server->client: Ubas3wModule |
| 14599 | `§_-83U§.as` | — |
| 14620 | `§_-g3n§.as` | server->client: ClientUISlotBarItemModule |
| 14641 | `§_-A4r§.as` | — |
| 14797 | `§_-D4F§.as` | — |
| 14864 | `§_-z14§.as` | — |
| 14871 | `§_-12a§.as` | — |
| 14974 | `§_-JR§.as` | — |
| 15000 | `§_-e1§.as` | — |
| 15045 | `§_-I3M§.as` | — |
| 15139 | `§_-Nm§_2.as` | — |
| 15267 | `§_-E1o§.as` | — |
| 15413 | `§_-J1k§.as` | — |
| 15451 | `§_-l2D§.as` | — |
| 15462 | `§_-i3y§.as` | server->client: OutOfBattleStationRangeCommand |
| 15473 | `§_-2P§.as` | — |
| 15545 | `§_-74B§.as` | — |
| 15665 | `§_-L1N§.as` | client->server: AssetHandleClickRequest |
| 15715 | `§_-U4l§.as` | — |
| 15758 | `§_-j2k§.as` | server->client: AssetTypeModule |
| 15763 | `§_-Dx§.as` | — |
| 15848 | `§_-k3f§.as` | — |
| 15997 | `§_-B2Z§.as` | server->client: AttributeHitpointUpdateCommand |
| 16033 | `§_-93a§_2.as` | — |
| 16131 | `§_-f4A§.as` | client->server: UnEquipModuleRequest |
| 16157 | `§_-q2H§.as` | — |
| 16172 | `§_-c1e§.as` | — |
| 16203 | `§_-I2C§.as` | — |
| 16258 | `§_-Y4r§.as` | — |
| 16380 | `§_-U2T§.as` | — |
| 16401 | `§_-k2e§.as` | — |
| 16435 | `§_-p2k§.as` | server->client: class_p2k |
| 16484 | `§_-82l§.as` | server->client: DisplaySettingsModule |
| 16508 | `§_-Z1M§.as` | — |
| 16633 | `§_-IJ§.as` | — |
| 16766 | `§_-kz§.as` | — |
| 16806 | `§_-u3u§.as` | — |
| 16993 | `§_-I4m§.as` | — |
| 17063 | `§_-z1U§.as` | client->server: MoveRequest |
| 17224 | `§_-r1P§.as` | — |
| 17372 | `§_-ox§.as` | — |
| 17423 | `§_-M1t§.as` | server->client: UbaM1tModule |
| 17430 | `§_-S3U§.as` | — |
| 17524 | `§_-G3F§.as` | server->client: UbaG3FModule |
| 17540 | `§_-v8§.as` | — |
| 17707 | `§_-d2h§.as` | server->client: HeroMoveCommand |
| 17857 | `§_-Y3D§.as` | — |
| 17873 | `§_-2H§.as` | — |
| 17926 | `§_-81r§.as` | — |
| 18090 | `§_-E2z§.as` | — |
| 18091 | `§_-V1a§.as` | client->server: ShipSelectRequest |
| 18140 | `§_-244§.as` | — |
| 18150 | `§_-M3R§.as` | — |
| 18193 | `§_-G4w§.as` | — |
| 18352 | `§_-7T§.as` | — |
| 18381 | `§_-k1T§.as` | server->client: UserSettingsCommand |
| 18383 | `§_-w3F§.as` | — |
| 18410 | `§_-f2e§.as` | — |
| 18425 | `§_-q1T§.as` | server->client: CreateBoxCommand |
| 18572 | `§_-61u§.as` | server->client: ClientUITooltipModule |
| 18603 | `§_-N4D§.as` | — |
| 18610 | `§_-Y3y§.as` | server->client: PetGearSelectCommand |
| 18631 | `§_-f3k§.as` | server->client: Ubaf3kModule |
| 18681 | `§_-K1I§.as` | client->server: SlotBarConfigSetRequest |
| 18777 | `§_-Bg§.as` | — |
| 18885 | `§_-8X§.as` | server->client: GroupChangeLeaderCommand |
| 18889 | `§_-a2x§.as` | client->server: SelectMenuBarItemRequest |
| 18943 | `§_-54m§.as` | server->client: PetGearTypeModule |
| 19068 | `§_-q2r§.as` | — |
| 19105 | `§_-Y39§.as` | — |
| 19121 | `§_-d28§.as` | — |
| 19125 | `§_-i4H§.as` | server->client: LevelUpCommand |
| 19134 | `§_-J10§.as` | server->client: DestructionTypeModule |
| 19147 | `§_-C4F§.as` | server->client: JumpCPUUpdateCommand |
| 19265 | `§_-e1U§.as` | server->client: BoostedAttributeTypeModule |
| 19280 | `§_-xU§.as` | server->client: GroupInvitationBehaviorModule<br>server->client: GroupUpdateInvitationBehaviorCommand |
| 19450 | `§_-C3A§.as` | — |
| 19501 | `§_-L3L§.as` | — |
| 19557 | `§_-jq§.as` | — |
| 19614 | `§_-D1c§.as` | — |
| 19797 | `§_-h14§.as` | server->client: ShipRemoveCommand |
| 19801 | `§_-E2Z§_2.as` | — |
| 19907 | `§_-I4f§.as` | server->client: AbilityEffectDeActivationCommand |
| 19929 | `§_-R4E§.as` | — |
| 20038 | `§_-G2O§.as` | client->server: QualitySettingsRequest |
| 20116 | `§_-Zo§.as` | server->client: GroupPlayerLocationModule |
| 20135 | `§_-k9§.as` | — |
| 20259 | `§_-qv§.as` | server->client: BattleStationErrorCommand |
| 20288 | `§_-I48§.as` | server->client: CreatePortalCommand |
| 20293 | `§_-l4g§.as` | — |
| 20306 | `§_-63B§.as` | — |
| 20320 | `§_-s1M§.as` | — |
| 20331 | `§_-Ji§.as` | — |
| 20472 | `§_-Q2w§.as` | — |
| 20489 | `§_-q1n§_2.as` | — |
| 20661 | `§_-8n§.as` | server->client: GroupPingCommand |
| 20767 | `§_-Cz§.as` | server->client: POIDesignModule |
| 20828 | `§_-N4A§.as` | — |
| 20985 | `§_-M3b§.as` | server->client: PlayGenericSoundCommand |
| 20988 | `§_-r3x§.as` | server->client: GameplaySettingsModule |
| 21036 | `§_-E4§.as` | — |
| 21125 | `§_-u30§.as` | server->client: UbaWindowInitializationCommand |
| 21137 | `§_-k2S§.as` | server->client: QualitySettingsModule |
| 21142 | `§_-tf§.as` | — |
| 21185 | `§_-a1L§.as` | — |
| 21233 | `§_-D26§.as` | server->client: UbaD26Module |
| 21236 | `§_-d1Q§.as` | — |
| 21243 | `§_-tr§.as` | — |
| 21351 | `§_-62j§.as` | — |
| 21421 | `§_-X1n§.as` | — |
| 21460 | `§_-1R§.as` | — |
| 21475 | `§_-u22§.as` | — |
| 21495 | `§_-I3z§.as` | server->client: BattleStationStatusCommand |
| 21499 | `§_-BE§.as` | — |
| 21555 | `§_-O2v§.as` | server->client: GroupUpdateBlockInvitationState |
| 21586 | `§_-A1e§.as` | — |
| 21587 | `§_-a3b§.as` | — |
| 21622 | `§_-r3C§.as` | — |
| 21683 | `§_-y2t§.as` | server->client: class_y2t |
| 21730 | `§_-o1v§.as` | — |
| 21788 | `§_-L3m§.as` | server->client: ClientUIMenuBarModule |
| 21825 | `§_-C2o§.as` | server->client: PetActivationCommand |
| 21885 | `§_-bl§.as` | — |
| 21931 | `§_-s1u§.as` | — |
| 21988 | `§_-82o§.as` | — |
| 22037 | `§_-o10§.as` | server->client: PetStatusCommand |
| 22038 | `§_-J1m§.as` | server->client: ShipDestroyedCommand |
| 22080 | `§_-h1g§.as` | — |
| 22112 | `§_-Y2b§.as` | — |
| 22136 | `§_-fS§.as` | — |
| 22208 | `§_-Of§.as` | — |
| 22230 | `§_-f1d§.as` | — |
| 22275 | `§_-w12§.as` | — |
| 22352 | `§_-e19§.as` | — |
| 22478 | `§_-G3a§.as` | — |
| 22742 | `§_-h1B§.as` | — |
| 22785 | `§_-e4S§.as` | — |
| 22801 | `§_-q2B§.as` | client->server: AttackRocketRequestHandler (Handler.cs) |
| 23083 | `§_-549§.as` | — |
| 23101 | `§_-Y§.as` | — |
| 23269 | `§_-C11§.as` | server->client: ClientUISlotBarCategoryItemModule |
| 23281 | `§_-61W§.as` | — |
| 23290 | `§_-l3g§.as` | — |
| 23311 | `§_-G3b§.as` | — |
| 23422 | `§_-T20§.as` | — |
| 23426 | `§_-F1f§.as` | — |
| 23438 | `§_-r2Z§.as` | — |
| 23446 | `§_-w1u§.as` | — |
| 23447 | `§_-y1L§.as` | — |
| 23506 | `§_-73u§.as` | — |
| 23530 | `§_-x23§.as` | — |
| 23543 | `§_-q3N§.as` | — |
| 23559 | `§_-P4T§.as` | server->client: ClientUISlotBarCategoryModule |
| 23579 | `§_-24d§.as` | — |
| 23586 | `§_-c3c§.as` | client->server: UIOpenRequest |
| 23600 | `§_-64j§.as` | — |
| 23618 | `§_-f1w§.as` | — |
| 23645 | `§_-o3w§.as` | — |
| 23663 | `§_-O25§.as` | — |
| 23727 | `§_-X4B§.as` | — |
| 23769 | `§_-S3o§.as` | — |
| 23900 | `§_-y3h§.as` | — |
| 23957 | `§_-X3h§.as` | — |
| 23977 | `§_-q34§.as` | — |
| 24015 | `§_-R1Y§.as` | — |
| 24024 | `§_-h3x§.as` | — |
| 24033 | `§_-i3w§.as` | — |
| 24146 | `§_-43r§.as` | — |
| 24193 | `§_-u§.as` | — |
| 24265 | `§_-qR§.as` | — |
| 24512 | `§_-V47§.as` | — |
| 24522 | `§_-R2s§.as` | — |
| 24653 | `§_-t1S§_2.as` | — |
| 24668 | `§_-Sg§.as` | — |
| 24674 | `§_-G4n§.as` | server->client: PetShieldUpdateCommand |
| 24735 | `§_-A1M§.as` | — |
| 24892 | `§_-P4w§.as` | server->client: ClientUITooltipTextFormatModule |
| 24942 | `§_-b4Y§.as` | — |
| 25052 | `§_-81f§.as` | — |
| 25086 | `§_-H4Q§.as` | server->client: class_H4Q |
| 25099 | `§_-EI§.as` | server->client: ShipDeselectionCommand |
| 25115 | `§_-a21§.as` | — |
| 25145 | `§_-D1p§.as` | — |
| 25158 | `§_-A3E§.as` | server->client: GroupRemoveCommand |
| 25203 | `§_-z2t§.as` | — |
| 25272 | `§_-i4W§.as` | — |
| 25300 | `§_-81I§.as` | client->server: GameplaySettingsRequest |
| 25310 | `§_-j13§.as` | — |
| 25425 | `§_-oS§.as` | server->client: class_oS |
| 25477 | `§_-d1e§.as` | server->client: DisposeBoxCommand |
| 25482 | `§_-L4v§.as` | — |
| 25570 | `§_-M23§.as` | — |
| 25672 | `§_-t2H§.as` | — |
| 25679 | `§_-23L§.as` | — |
| 25842 | `§_-g2Y§.as` | — |
| 25891 | `§_-bc§.as` | server->client: GroupPlayerInformationsModule |
| 25971 | `§_-gB§.as` | client->server: KillscreenRequest |
| 25977 | `§_-c3f§.as` | — |
| 26002 | `§_-E30§.as` | — |
| 26054 | `§_-J1h§.as` | — |
| 26075 | `§_-S22§.as` | — |
| 26150 | `§_-k2B§.as` | — |
| 26197 | `§_-Q3Y§.as` | server->client: ClanChangedCommand |
| 26269 | `§_-o21§.as` | — |
| 26416 | `§_-04r§.as` | — |
| 26571 | `§_-53A§.as` | client->server: GroupUpdateBlockInvitationStateRequestHandler (Handler.cs) |
| 26601 | `§_-Os§_2.as` | — |
| 26611 | `§_-O4f§.as` | server->client: class_O4f |
| 26664 | `§_-G4F§.as` | — |
| 26667 | `§_-s16§.as` | server->client: class_s16 |
| 26745 | `§_-TJ§.as` | — |
| 26764 | `§_-Sw§.as` | — |
| 26941 | `§_-12Y§.as` | client->server: EmergencyRepairRequest |
| 26950 | `§_-I2Z§.as` | server->client: BoosterTypeModule |
| 27151 | `§_-V4y§.as` | — |
| 27245 | `§_-c2V§.as` | server->client: GroupPlayerInCombatModule |
| 27259 | `§_-R4n§.as` | — |
| 27262 | `§_-ur§.as` | server->client: AudioSettingsModule |
| 27279 | `§_-f1Q§.as` | — |
| 27317 | `§_-r3R§.as` | — |
| 27329 | `§_-JN§.as` | — |
| 27378 | `§_-pJ§.as` | server->client: ClanRelationModule |
| 27509 | `§_-E1W§.as` | — |
| 27527 | `§_-l3I§.as` | — |
| 27619 | `§_-71b§.as` | server->client: GroupPlayerShipModule |
| 27641 | `§_-z3Q§.as` | server->client: Ubaz3QModule |
| 27666 | `§_-p1x§.as` | — |
| 27856 | `§_-G3g§.as` | — |
| 27873 | `§_-i3h§.as` | — |
| 27912 | `§_-RS§.as` | server->client: MapRemovePOICommand |
| 28000 | `§_-13m§.as` | — |
| 28077 | `§_-22M§.as` | — |
| 28127 | `§_-EL§.as` | server->client: AttackLaserRunCommand |
| 28142 | `§_-64b§.as` | — |
| 28186 | `§_-K3i§.as` | — |
| 28243 | `§_-Xg§.as` | — |
| 28351 | `§_-u3O§.as` | server->client: PetGearAddCommand |
| 28373 | `§_-Ht§.as` | server->client: UbaHtModule |
| 28480 | `§_-B2I§.as` | — |
| 28484 | `§_-K1J§.as` | — |
| 28501 | `§_-mJ§.as` | — |
| 28685 | `§_-qe§.as` | client->server: GroupChangeGroupBehaviourRequestHandler (Handler.cs) |
| 28690 | `§_-h1b§_2.as` | — |
| 28700 | `§_-729§.as` | — |
| 28720 | `§_-O4k§.as` | — |
| 28738 | `§_-12e§.as` | — |
| 28800 | `§_-v1j§.as` | — |
| 28841 | `§_-S1o§.as` | — |
| 28872 | `§_-j4x§.as` | — |
| 29001 | `§_-Z3Z§.as` | — |
| 29002 | `§_-X13§.as` | — |
| 29105 | `§_-1i§.as` | server->client: GroupPlayerModule |
| 29107 | `§_-C2b§.as` | — |
| 29161 | `§_-92t§.as` | client->server: GroupKickPlayerRequest |
| 29238 | `§_-k19§.as` | server->client: GroupPlayerTargetModule |
| 29319 | `§_-j17§.as` | — |
| 29441 | `§_-dM§.as` | server->client: POITypeModule |
| 29508 | `§_-f2N§.as` | — |
| 29541 | `§_-p3t§.as` | — |
| 29719 | `§_-cU§.as` | — |
| 29761 | `§_-LU§.as` | — |
| 29819 | `§_-V3§.as` | server->client: MoveCommand |
| 29823 | `§_-go§.as` | server->client: PetHitpointsUpdateCommand |
| 29864 | `§_-F2I§.as` | server->client: class_F2I |
| 29943 | `§_-34h§.as` | — |
| 29946 | `§_-i3K§.as` | — |
| 30023 | `§_-h2P§.as` | server->client: class_h2P |
| 30075 | `§_-d3F§.as` | — |
| 30106 | `§_-V4m§.as` | server->client: ClientUISlotBarsCommand |
| 30161 | `§_-84z§.as` | — |
| 30352 | `§_-j3V§.as` | — |
| 30475 | `§_-T14§.as` | — |
| 30515 | `§_-24J§.as` | — |
| 30647 | `§_-Z3F§.as` | — |
| 30651 | `§_-i4x§.as` | — |
| 30741 | `§_-I2b§.as` | server->client: ClientUISlotBarModule |
| 30787 | `§_-r1l§.as` | server->client: MapAssetActionAvailableCommand |
| 30831 | `§_-io§.as` | — |
| 30907 | `§_-A2J§.as` | client->server: WindowSettingsRequest |
| 30931 | `§_-U1I§.as` | — |
| 30947 | `§_-b1O§.as` | server->client: BattleStationManagementUiInitializationCommand |
| 30963 | `§_-94§.as` | server->client: GroupPlayerDisconnectedModule |
| 30964 | `§_-x1v§.as` | — |
| 31038 | `§_-r1N§.as` | — |
| 31080 | `§_-Pk§.as` | — |
| 31092 | `§_-o2v§_2.as` | server->client: KillScreenOptionModule |
| 31106 | `§_-T31§.as` | client->server: AttackLaserRequestHandler (Handler.cs) |
| 31113 | `§_-62J§_2.as` | server->client: RemovePortalCommand |
| 31208 | `§_-dj§.as` | server->client: ClientUISlotBarCategoryItemTimerModule |
| 31215 | `§_-9z§.as` | — |
| 31269 | `§_-01K§.as` | — |
| 31291 | `§_-Q3b§.as` | — |
| 31344 | `§_-C3T§.as` | server->client: RemoveMenuItemHighlightCommand |
| 31644 | `§_-012§.as` | — |
| 31655 | `§_-I3y§_2.as` | — |
| 31697 | `§_-XM§.as` | client->server: ProActionBarRequest |
| 31812 | `§_-S3v§.as` | server->client: CpuInitializationCommand |
| 31849 | `§_-A1Z§.as` | client->server: PetGearActivationRequest |
| 31903 | `§_-C3l§.as` | — |
| 31946 | `§_-442§.as` | — |
| 31959 | `§_-C3D§.as` | — |
| 31968 | `§_-I1q§.as` | — |
| 31992 | `§_-533§.as` | server->client: class_533 |
| 32052 | `§_-u2M§.as` | — |
| 32054 | `§_-T1K§.as` | — |
| 32081 | `§_-K3s§.as` | — |
| 32090 | `§_-G40§.as` | server->client: GroupPlayerHadesGateModule |
| 32194 | `§_-F4j§.as` | — |
| 32205 | `§_-K1b§.as` | server->client: AttackTypeModule |
| 32334 | `§_-q2h§_2.as` | server->client: PetRepairCompleteCommand |
| 32339 | `§_-i3O§.as` | — |
| 32358 | `§_-513§.as` | — |
| 32380 | `§_-N22§.as` | client->server: GroupPingPlayerRequest |
| 32409 | `§_-T3n§.as` | server->client: PetHeroActivationCommand |
| 32428 | `§_-o3q§.as` | server->client: class_o3q |
| 32484 | `§_-l3T§.as` | — |
| 32621 | `§_-N2H§.as` | client->server: AudioSettingsRequest |

## Cómo interpretar los nombres de clase C#

| Prefijo o patrón | Rol típico |
|------------------|------------|
| `Attack*`, `Attribute*`, `Ability*` | Combate, impactos, escudo/vida, tipos de ataque |
| `Pet*` | PET: estado, equipo, reparación, XP, activación |
| `Group*` | Grupo: invitaciones, ping, liderazgo, módulos por jugador |
| `Ship*` | Creación/destrucción/selección de naves en mapa |
| `ClientUI*` | Barras rápidas, tooltips, menús, categorías de slots |
| `Map*`, `Asset*`, `POI*` | POIs, assets del mapa, portales |
| `BattleStation*` | UI y estado de estación de batalla de clan |
| `UserSettings*`, `Quality*`, `Display*`, `Audio*`, `Gameplay*`, `Window*`, `*KeyBindings*` | Ajustes persistidos (login / opciones) |
| `MoveCommand`, `HeroMoveCommand` | Sincronización de movimiento (entidades vs héroe) |
| `Mine*`, `CreateBox*`, `DisposeBox*` | Minas y cajas/recolectables |
| `JumpCPU*`, `Portal*`, `ActivatePortal*` | Salto / portales |
| `KillScreen*`, `LevelUp*` | Pantalla de muerte, subida de nivel |
| `class_*` | Paquetes alineados con nombres ofuscados del cliente; revisar el `.cs` para campos |
| `Uba*` (módulos) | Datos embebidos en flujos UBA (ventanas / matchmaking) |

## Archivos relevantes en el servidor

| Ruta | Función |
|------|---------|
| `Net/netty/Handler.cs` | Despacho de peticiones entrantes (`Dictionary<short, IHandler>` + IDs literales) |
| `Net/netty/commands/*.cs` | Escritura de paquetes salientes (`ID` + `write(...)`) |
| `Net/netty/requests/**/*.cs` | Lectura de peticiones entrantes (`readCommand`) |
| `Net/GameClient.cs` | Recibe bytes y delega en `Handler.Execute` |
| `Net/netty/handlers/*.cs` | Lógica de negocio por petición |

## Datos generados

- `server-protocol-ids.csv`: todas las parejas `(Id, Direction, Class)` extraídas de `commands` y `requests`.
- `g4v-server-crossref.csv`: mismo cruce que esta tabla en CSV.
