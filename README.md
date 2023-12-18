# MyFirstrepo
//
//  MediaPickerView.swift
//  CollageBuilder
//
//

import SwiftUI
import PhotosUI

struct MediaPickerView: UIViewControllerRepresentable {
    
    @Binding var media: Media?
    
    func makeUIViewController(context: Context) -> PHPickerViewController  {
        var config = PHPickerConfiguration()
        config.selectionLimit = 1
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: UIViewControllerType, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        return Coordinator(media: $media)
    }
    
    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        
        @Binding var media: Media?
        
        init(media: Binding<Media?>) {
            self._media = media
        }
        
        func picker(_ picker: PHPickerViewController,
                    didFinishPicking results: [PHPickerResult]) {
            
            guard let provider = results.first?.itemProvider else {
                picker.dismiss(animated: true)
                return
            }
            
            if provider.canLoadObject(ofClass: UIImage.self) {
                provider.loadObject(ofClass: UIImage.self) { image, _ in
                    guard let image = image as? UIImage else { return }
                    Task { @MainActor in
                        self.media = .init(resource: .image(image))
                        picker.dismiss(animated: true)
                    }
                }
                
            } else if provider.hasItemConformingToTypeIdentifier(UTType.movie.identifier) {
                let _ = provider.loadTransferable(type: Video.self) { result in
                    guard case .success(let video) = result else { return }
                    Task { @MainActor in
                        self.media = .init(resource: .video(video))
                        picker.dismiss(animated: true)
                    }
                }
                
            } else {
                picker.dismiss(animated: true)
            }
        }
    }
}
