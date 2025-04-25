# DeterminatedSizeAlloc
Alloc for ObjectPascal with determinated size.

Memory allocation, with the ability to determine the size of the allocated memory block.  
You can apply the same idea to other languages ​​and environments.  

In addition, to protect against Use After Free and similar errors, allocated memory is cleared both after allocation and after freeing.  

* `function VmAllocMemory(Size: NativeInt): Pointer;` ~ Alloc memory
* `function VmReallocMemory(Memory: Pointer; Size: Integer): Pointer;` ~ Realloc memory
* `function VmFreeMemory(Memory: Pointer): Pointer;` ~ Free memory
* `function VmSizeMemory(Memory: Pointer): NativeInt;` ~ Determine size of memory block

If it was not possible to allocate or reallocate memory, the functions return nil, FUNCTIONS DO NOT THROW OOM EXCEPTION.  
Already allocated memory remains valid.  

### Source code:
```Pascal
{$IFNDEF FPC}
type
  TMemoryManager = TMemoryManagerEx;
{$ENDIF}

function VmAllocMemory(Size: NativeInt): Pointer;
var
  MemoryManager: TMemoryManager;
begin
  // get current memory manager
  GetMemoryManager(MemoryManager);

  // check zero
  if Size <= 0 then
  begin
    exit(nil);
  end;

  // try alloc mem
  Result := MemoryManager.GetMem(Size + SizeOf(NativeInt));

  // check out of memory
  if Result = nil then
  begin
    exit(nil);
  end;

  // save size
  PNativeInt(Result)^ := Size;

  // inc Result
  Inc(PNativeInt(Result));

  // fill zeros
  FillChar(Result^, Size, 0);
end;

function VmReallocMemory(Memory: Pointer; Size: Integer): Pointer;
var
  MemoryManager: TMemoryManager;
  StartFill: PByte;
  OldSize: NativeInt;
begin
  // get current memory manager
  GetMemoryManager(MemoryManager);

  // check just alloc
  if Memory = nil then
  begin
    Result := VmAllocMemory(Size);
    exit;
  end;

  // check just free
  if Size <= 0 then
  begin
    // just free
    Result := VmFreeMemory(Memory);
    exit;
  end;

  // dec Memory
  Dec(PNativeInt(Memory));

  // get old size
  OldSize := PNativeInt(Memory)^;

  // zeros for cut mem
  if Size < OldSize then
  begin
    StartFill := Memory;
    Inc(StartFill, SizeOf(NativeInt));
    Inc(StartFill, Size);
    FillChar(StartFill^, OldSize - Size, 0);
  end;

  // try realloc
  Result := MemoryManager.ReallocMem(Memory, Size + SizeOf(NativeInt));

  // check out of memory
  if Result = nil then
  begin
    exit(nil);
  end;

  // save size
  PNativeInt(Result)^ := Size;

  // inc Result
  Inc(PNativeInt(Result));

  // fill zeros
  StartFill := Result;
  Inc(StartFill, OldSize);
  FillChar(StartFill^, Size - OldSize, 0);
end;

function VmFreeMemory(Memory: Pointer): Pointer;
var
  MemoryManager: TMemoryManager;
  StartFill: PByte;
  Size: NativeInt;
begin
  // check nil mem
  if Memory = nil then
  begin
    exit(nil);
  end;

  // get current memory manager
  GetMemoryManager(MemoryManager);

  // dec Memory
  Dec(PNativeInt(Memory));

  // fill zeros
  Size := PNativeInt(Memory)^;
  StartFill := Memory;
  Inc(StartFill, SizeOf(NativeInt));
  FillChar(StartFill^, Size, 0);

  // free
  MemoryManager.FreeMem(Memory);
  Result :=  nil;
end;

function VmSizeMemory(Memory: Pointer): NativeInt;
begin
  if Memory = nil then
  begin
    exit(0);
  end;

  // dec
  Dec(PNativeInt(Memory));

  // get size
  Result := PNativeInt(Memory)^;
end;
```


