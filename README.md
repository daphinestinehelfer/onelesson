# onelesson
//
//  StickerEditorView.swift
//  CollageBuilder
//
//

import SwiftUI
import CoreImage.CIFilterBuiltins

struct StickerEditorView: View {
    
    @EnvironmentObject private var store: AppStore
    @State private var showMaskEditor = false
    
    var body: some View {
        Section("Common settings") {
            commonSettings
        }
        
        Section("mask") {
            editMask
        }
        
        Section("manage") {
            remove
        }
        .sheet(isPresented: $showMaskEditor) {
            if let sticker {
                StickerEraserView(sticker: sticker)
                    .presentationBackground(.thinMaterial)
            }
        }
        
        Section("Animation") {
            animation
        }
    }
    
    private var animation: some View {
        AnimationSelectorView(animation: .init(
            get: { sticker?.animation },
            set: { dispatch(.changeAnimation($0)) }
        ))
    }
    
    private var commonSettings: some View {
        VStack {
            BlendModeSelectorView(blendMode: .init(
                get: { sticker?.blendMode ?? .normal },
                set: { dispatch(.changeBlendMode($0)) }
            ))
            ZPositionSelectorView(zPosition: .init(
                get: { sticker?.zPosition ?? 0 },
                set: { dispatch(.changeZPosition($0)) }
            ))
        }
    }
    
    private var editMask: some View {
        HStack {
            Text("Edit mask")
            Spacer()
            Button {
                showMaskEditor.toggle()
            } label: {
                Image(systemName: "paintbrush.pointed.fill")
                    .font(.title2)
            }
        }
    }
    
    private var remove: some View {
        HStack {
            Text("Remove")
            Spacer()
            Button {
                if let sticker {
                    store.dispatch(.changeCollage(
                        .removeSticker(sticker.id)
                    ))
                }
            } label: {
                Image(systemName: "trash.slash")
                    .font(.title2)
            }
        }
    }
    
    private var sticker: Sticker? {
        let sticker = store.state.collage.stickers.first(where: {
            $0.id == store.state.selectedElement?.stickerId
        })
        
        return sticker
    }
    
    private func dispatch(_ action: StickerModification) {
        guard let id = store.state.selectedElement?.stickerId else {
            return
        }
        
        store.dispatch(.changeCollage(.changeSticker(action, id: id)))
    }
    
}

struct StickerEditorView_Previews: PreviewProvider {
    static var previews: some View {
        List {
            StickerEditorView()
                .environmentObject(AppStore.preview)
        }
    }
}
